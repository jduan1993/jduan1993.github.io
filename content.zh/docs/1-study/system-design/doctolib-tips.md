---
title: Doctolib 大纲
weight: 2
---

### 用途
#### 1. API Gateway
- 身份验证（Authentication & Authorization）
- 速率限制（Rate Limit）
- 请求跟踪（Request Tracing）
- 熔断（Circuit Breaking）
#### 2. Search Service
- Redis + Database搜索
#### 3. Booking Service
- Redis Redlock，时段级锁定（Slot-level Locking），数据库事务（Transaction）
- Idempotency，Request ID
- 发布到Kafka进行缓存失效和通知
##### 预约流程
- Redlock加锁
- 数据库悲观锁（Pessimistic Lock）双检
- 插入预约（Appointment），加入Request ID作为唯一性约束
- 发布AppointmentCreated到Kafka
- 释放锁（Unlock）
- 响应客户端
- 消费者（Search Service）Cache Invalidation
- 消费者（Notification Service）发送消息
#### 4. Database
- 数据库分区（Partition by country or Doctor‘s ID）
- 消息总线（Message Bus），用于解耦预约和缓存失效和通知
#### 5. Notification Service
- Email Service or SMS Service （Sendgrid & Twilio）
- In-app notification （Firebase etc.)

### 扩展
- 根据CPU，Memory等进行横向拓展（Horizontal Pod Autoscaler）
- redis分片
- 数据库读写分离（Read-Write Separation）
- 数据库分区（Partition）

### 容错
- 熔断（Circuit Breaker， Resilience4j）
- 健康检查（Health Checks）
- 自我修复（Self-Healing）
- 告警（Alert）

### 安全
- 端到端TLS
- OAuth2登录
- Rate Limit
- Logging

### 面试表述
#### 负载均衡和熔断限流在哪里做，不在gateway做吗？
- “我们在 Gateway 层做外部限流和初级熔断，用云厂商 LB + Ingress/Gateway 控制边缘流量；在 服务内部，客户端用 Ribbon 或 Envoy 做调用负载均衡，用 Resilience4j 做熔断和限流保护下游；在 平台侧，Kubernetes Service 负责 Pod 级别均衡，若引入 Service Mesh，则 Envoy sidecar 可做更细粒度的 LB、熔断、限流、流量镜像等高级功能。这样多层协同，既保护了下游服务，也保证了全链路的高可用和可控性。”
#### k8s中推荐用什么方式做用户侧调用和服务间调用的负载均衡、限流、熔断？
- “在边缘我们用云端 LB + Ingress 或 Envoy-based Gateway 做外部负载均衡和限流，可选性强且搭配插件化熔断；在服务内部调用层面，推荐用 Service Mesh（Envoy Sidecar）来透明做服务发现、客户端负载均衡、网络限流和熔断，代码层面再用 Resilience4j 对关键依赖做二次保护。”
#### k8s需要做注册中心集群吗？
- “在 Kubernetes 中，API Server + etcd + kube-proxy 本身就构成了高可用的服务注册与发现体系，我们只需创建 Service 资源，K8s 自动维护 Endpoints 并做 DNS/负载均衡。除非有跨集群或必须用 Spring Cloud Netflix 生态的遗留需求，否则不再额外部署注册中心集群，这样能简化架构并利用 K8s 的原生 HA 能力。”

### 待解决问题
#### 如果快速将服务扩展到其他国家？
##### 1. 目标拆解：支持多国家的能力需求
| 维度        | 要求说明                       |
| --------- | -------------------------- |
| 🏛 数据隔离   | 每个国家业务数据逻辑隔离（合规/监管要求）      |
| 🌐 业务配置差异 | 国家A/B 的工作日、货币、语言、时间等不同     |
| 🧰 服务逻辑差异 | 部分微服务逻辑可能略有差异              |
| 🚀 快速部署   | 新国家上线不需大改动，支持可配置部署         |
| 🔐 合规安全   | 数据主权（Data Residency）遵守各国法规 |
##### 2. 配置驱动的多国家支持
| 方法               | 实现                                            |
| ---------------- | --------------------------------------------- |
| 配置中心             | 每个国家一份配置，如 `config_fr.yaml`, `config_de.yaml` |
| 通过配置注入           | 控制：开放时间段、货币、支持语言、特殊逻辑开关                       |
| 多租户标识（Tenant ID） | 请求中传入国家 ID (`X-Country: FR`)，统一识别             |
