---
title: 系统设计
weight: 1
---

### 要求
#### 1. 分布式、可扩展。
- 分布式：微服务、异步通信、分布式缓存、K8s
- 可扩展：HPA、分库分表、K8s DNS Service、多AZ、多国家
#### 2. 用户可搜索、可预约、可重新安排、可取消，基于医生的位置、专科、可用时间。
##### 搜索
- Cache Aside，Invalidate when Booking and Cancellation (TTL ~ minutes)
- Caffeine 异步延迟双删
###### 权衡
| 优点      | 说明                          |
| ------- | --------------------------- |
| ⚡ 高吞吐   | Redis 命中率高，支持 QPS > 1000+   |
| ⏱️ 低延迟  | 单医生查询 latency ≈ 几 ms        |
| ✅ 写后无读  | Slot 数据预生成、写入后只读，适合缓存       |
| 🔁 异步更新 | Cache 由 Kafka 驱动异步更新，无需强一致性 |

| 缺点                     | 说明                                      |
| ---------------------- | --------------------------------------- |
| 🕒 数据轻微不一致             | TTL 失效期间可能展示已被预约的 slot（在 Booking 阶段兜底）  |
| 🧠 Cache Invalidate 复杂 | 多服务写入 slot 状态需保证正确清除 Redis 缓存           |
| 📦 大量 slot             | Redis 内存占用需控制（可做分片 or 使用 Redis Cluster） |

##### 预约
- Redlock 加锁 (TTL ~ seconds)
- 加入 Request ID 作为唯一性约束
- 数据库悲观锁（Pessimistic Lock）双检
- 插入预约（Appointment）
- 发布 AppointmentCreated 到 Kafka
- 释放锁（Unlock）
- 响应客户端
- 消费者（Search Service）Cache Invalidation
- 消费者（Notification Service）发送消息
###### 权衡
| 优点       | 说明                   |
| -------- | -------------------- |
| ✅ 高一致性保障 | 锁+事务双重保证             |
| ✅ 幂等设计   | 防止重复预约               |
| ✅ 可观测性强  | Kafka 推送事件用于追踪链路     |
| ✅ 异步解耦   | 非核心逻辑（通知、缓存）不影响主流程性能 |

| 缺点          | 说明                                         |
| ----------- | ------------------------------------------ |
| ❗ Redis 不可用 | Redlock 依赖 Redis，可引入降级机制（fallback DB lock） |
| ❗ 高并发下延迟    | 拥塞在热点 slot（比如热门医生）可用队列排队缓解                 |
| ❗ 事务复杂性     | 需细致处理锁释放和回滚，避免死锁                           |

##### 重新安排
- Redlock加锁 (TTL ~ seconds)
- 数据库悲观锁（Pessimistic Lock）双检
- 校验修改旧预约，插入新预约
- 发布 AppointmentRescheduled 到 Kafka
- 释放锁（Unlock）
- 响应客户端
- 消费者（Search Service）Cache Invalidation
- 消费者（Notification Service）发送消息
###### 权衡
| 优点            | 说明                     |
| ------------- | ---------------------- |
| ✅ 一致性保障       | 事务更新 + 幂等控制，避免错约/重复约   |
| ✅ 双 slot 安全管理 | 显式释放旧 slot、锁定新 slot    |
| ✅ 解耦逻辑        | Kafka 用于通知与缓存刷新，主流程不阻塞 |

| 缺点                       | 说明                 |
| ------------------------ | ------------------ |
| ❗ 新旧 slot 状态切换逻辑复杂       | 需要明确状态流转模型         |
| ❗ Redis 不可用影响锁           | 降级 fallback：数据库悲观锁 |
| ❗ 用户误操作可能导致重复 reschedule | 需限制频率，增强幂等性保障      |

##### 取消
- Redlock加锁 (TTL ~ seconds)
- 数据库悲观锁（Pessimistic Lock）双检
- 校验修改旧预约
- 发布 AppointmentCanceled 到 Kafka
- 释放锁（Unlock）
- 响应客户端
- 消费者（Search Service）Cache Invalidation
- 消费者（Notification Service）发送消息
###### 权衡
#### 3. 服务5000万用户，50万医生，每秒700次搜索，每秒90次预约。
#### 4. 轻松拓展新地区。
- 配置层面隔离（Spring Cloud Config）
- 数据库分库（Date Partitioning）
- 通用服务（Common Services）和差异化定制（Custom Services）
- API设计（/fr, /de）
- 多租户部署（namespace）
- 日志标签（Label）
#### 5. 要考虑服务之间的扩展性，容错性，数据一致性。
- 扩展性：HPA，Ingress Controller，Redis Cluster Slot，Kafka Brokers Partition, Database Sharding
- 容错性：Service Mesh，多AZ，Redis Redlock，Kafka
- 最终一致性：数据库事务，Redis Redlock，SAGA Pattern，异步消息（Async Message），重试（Retry），死信队列（Dead Letter Queue），缓存失效（Cache Invalidation），事件监听（Event-driven Monitoring）

### 问题
#### 1. 搜索前（Search）需要经过鉴权（Authorization）吗？
如果不需要：
- 把Search Service置前
- 这样可能更吸引用户
- 但是会增加访问量
#### 2. 搜索时都有什么筛选条件（Filter Condition）？
如果涉及到距离，交通工具，路程用时等条件： 
- 可以设计第三方供应商（3rd-Party Vendors）提供地图（Map API）等接口

如果涉及到医生简介（Summary），全文搜索（Full-Text Search）:
- 可以引入ElasticSearch
#### 3. 我们不同国家的市场使用同一个应用（The only one app）吗？
如果是：
- 国际化（Internationalization），时间、语言、货币、日期格式
- 配置开关（Feature Flag）
- 根据设备本地时间（Device Time）或者定位（Localization）

如果不是：
- 部署过程要考虑更多模块
#### 4. 预约会跨天（Span 2 days）吗？
- 要考虑数据库表结构（Database table structure)
- 代码逻辑中要考虑
#### 5. 每秒700次搜索和90次预约是最高峰值（Maximum Peak Value）吗？
- 影响规模（Scale）

### 优化
- 服务间通信双向TLS（Mutual TLS between services）
- 用Service Mesh统一做熔断、限流、重试（Centralized resiliency by service mesh）
- 多AZ部署，包括Services，Redis Cluster等（Multi-AZ）
- 二级缓存，一级Caffeine（Local），二级Redis（Distributed）
- 数据库读写分离（Read-Write Separation），分库分表（Sharding）
- 全链路Trace（Telemetry，end-to-end tracing）
- 固定时间批处理热点时间多（Scheduled batch to store hot time slots）

