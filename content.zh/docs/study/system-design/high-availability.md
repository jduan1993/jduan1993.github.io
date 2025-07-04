---
title: 高可用性
---

# 服务宕机与网络分区时保证系统可用性 & CAP 原则解析

---

## 一、服务宕机或网络分区时如何保证系统整体可用？

### 1. 多副本冗余部署

* 对关键服务部署多实例，分布在不同节点或可用区，防止单点故障导致服务不可用。
* 通过负载均衡自动切换到健康实例。

### 2. 健康检查与自动故障转移

* 利用健康探针（Liveness/Readiness Probe）检测服务状态，异常自动剔除。
* K8s、服务网格等支持自动流量路由调整。

### 3. 限流、熔断和降级

* 防止故障传播，减少故障扩散范围，保障核心服务可用。
* 在部分依赖不可用时，提供降级功能维持核心体验。

### 4. 数据副本与异步复制

* 保证数据多副本存储，避免单点数据丢失。
* 异步复制减少同步阻塞，提高可用性，但存在一定数据一致性风险。

### 5. 设计幂等和重试机制

* 保障请求安全重试，避免因网络抖动导致的数据错误。

---

## 二、CAP 原则简介

CAP 定理指出，分布式系统不能同时完美满足：

* **C（一致性 Consistency）**：所有节点读到的数据是一致的，最新的写入立即可见。
* **A（可用性 Availability）**：每个请求都能得到响应（成功或失败），系统对所有节点持续可用。
* **P（分区容忍性 Partition Tolerance）**：系统能在网络分区（节点间消息丢失或延迟）时继续运行。

由于网络分区不可避免，系统设计时需在**一致性**和**可用性**之间权衡。

---

## 三、实际系统中的 CAP 取舍

| 取舍方向           | 说明                            | 典型场景或技术                |
| -------------- | ----------------------------- | ---------------------- |
| **CP（优先一致性）**  | 保证数据强一致性，分区时牺牲可用性（部分请求可能失败）   | 传统关系型数据库、分布式事务系统       |
| **AP（优先可用性）**  | 保证系统持续响应，允许短暂数据不一致，后续进行数据同步修正 | 大规模互联网应用、缓存系统、部分 NoSQL |
| **CA（理论上的理想）** | 不考虑网络分区，只在单节点或强同步场景下能实现       | 单机系统或网络稳定环境            |

---

## 四、结合场景的实践建议

* **核心业务（如支付、订单）**：更倾向于**CP**，确保数据准确，允许部分请求失败或延迟。
* **用户体验相关（如浏览、推荐）**：可偏向**AP**，保证响应速度和可用性，允许短暂的数据不一致。
* **采用异步补偿和最终一致性设计**：用 Saga 模式等实现业务流程的补偿，提升系统可用性。

---

## 五、总结

| 关注点     | 设计策略                        |
| ------- | --------------------------- |
| 宕机/分区容忍 | 多副本冗余、健康检查、自动故障转移、降级限流      |
| CAP权衡   | 根据业务优先级选CP或AP，设计幂等与补偿机制     |
| 系统弹性    | 利用异步机制与最终一致性，提升可用性同时保证数据正确性 |

---