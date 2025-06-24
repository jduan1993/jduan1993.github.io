---
title: Kafka Exactly-Once
---

# 流程

* 幂等生产者（生产者 Producer ID + 分区维护 Sequence Number）
* 事务协调者带 _transaction_state（用户 Transaction ID + 生产者 epoch + 分区 Control Messages + 消费者 LSO）
* 完整流程：
1. 事务初始化
   * 生产者寻找事务协调者（Transactional Coordinator）。
   * 生产者向事务协调者获取  PID。
2. 事务开始
3. Consume-Process-Produce：
   * 消费者（流处理应用同时作为消费者和生产者）从源 Topic 消费消息并做处理。
   * 生产者同步消息所要发往的  Topic-Partition 信息给事务协调者。
   * 生产者向目标 Topic-Partition 发送消息。
   * 生产者同步提交位点所要发往的  Topic-Partition （内部 Topic __consumer_offsets）信息给事务协调者。
   * 生产者通知 消费组协调者（Group Coordinator，服务端负责感知消费组变化的 Broker）提交位点（仅持久化位点，但并 未更新缓存，因此直到事务提交前对消费者不可见）
4. 提交/回滚事务
   * 生产者告知事务协调者事务 执行结果（提交/回滚）。
   * 事务协调者向事务所涉及的分区 Leader 发送 控制消息标记事务执行结果，同时事务协调者给生产者响应事务已提交/回滚成功。
   * 待所有分区 Leader 将控制消息持久化（任何一个失败都会进行无限重试）后，事务协调者将该事务状态修改为已提交/回滚，事务结束。
   
---

下面基于上述 Exactly-Once 语义实现的内容，对“重要场景”与“关键设计”分别进行归纳与解释。

---

# 归纳与解释

## 1. 重要场景及 Kafka 在其中的角色

1. **At-Most-Once / At-Least-Once vs Exactly-Once 流处理场景对比**

   * **场景说明**：流处理通常涉及“消费-处理-生产”（Consume-Process-Produce）流程；若仅用 At-Least-Once 语义，会因网络抖动或进程崩溃导致消息重复处理；若 At-Most-Once，则可能丢失消息。
   * **Kafka 角色**：提供端到端 Exactly-Once 能力（在消费来源于 Kafka，且输出写回 Kafka 的场景）。通过幂等生产者与事务机制，避免重复消息与丢失，确保每条输入消息“恰好一次”被处理并产出消息，同时提交位点也被原子化地与输出挂钩。

2. **网络 ACK 丢失导致重试的重复写入场景**

   * **场景说明**：生产者发送消息，Broker 实际已写入但 ACK 丢失，客户端误以为失败重试，导致重复写入。
   * **Kafka 角色**：在幂等生产者中，通过 Producer ID + 分区内序列号、服务端维护最近 N 条批次信息，判重与乱序检测；再结合事务可保证跨会话的幂等，避免因 ACK 丢失产生重复。

3. **消费端已发送输出但尚未提交位点前崩溃导致重读并重复写入场景**

   * **场景说明**：在流处理应用中，消息处理完并向目标 Topic 发送成功后，若在提交 consumer offset 之前崩溃，重启后会再次消费并重复发送。
   * **Kafka 角色**：通过事务：把“发送消息到目标 Topic”与“提交源 Topic 的位点（\_\_consumer\_offsets）”都纳入同一事务。若事务最终提交，则两者一起生效；若中途失败或崩溃未提交，则事务回滚，输出消息不可见且位点也不会生效，从而避免重复计算。

4. **多分区或多次读写操作的原子性需求场景**

   * **场景说明**：流处理可能对多个分区写入，或同时需要向多个 Topic-Partition 发送结果，并提交多个操作（如写结果、更新状态 topic、提交位点等）。需要保证这些操作要么都生效，要么都不生效。
   * **Kafka 角色**：引入事务协调者（Transaction Coordinator），在事务上下文中跟踪涉及的所有分区信息；最终通过控制消息（WriteTxnMarker）通知各分区 Leader 提交或回滚，确保原子性。

5. **跨会话恢复场景**

   * **场景说明**：流处理应用或生产者进程重启后，若使用相同的事务 ID，应能识别前一会话未完成的事务并进行恰当回滚或继续。
   * **Kafka 角色**：Transactional ID（用户提供）+ 服务端分配的 PID + epoch 机制。重启后客户端用相同 Transactional ID 找到协调者，协调者检测到旧事务未完成，会回滚旧事务，给新的会话分配新的 epoch，从而保证幂等与正确恢复。

6. **Broker/协调者故障或网络分区场景**

   * **场景说明**：事务协调者或分区 Leader 崩溃、切换；网络暂时不可达等。需要确保事务能在故障后继续或回滚，不会出现混乱或数据丢失。
   * **Kafka 角色**：

     * **事务协调者 HA**：协调者元数据持久化于内部 \_\_transaction\_state compacted Topic，多副本复制保障；故障切换后，新协调者通过读取日志恢复状态并继续发送控制消息或回滚。
     * **分区 Leader HA**：PID Snapshot、日志恢复机制，让服务端恢复 Producer State 映射，继续判重与乱序检测；WriteTxnMarker 重试机制确保控制消息最终写入。

7. **长事务与 Last Stable Offset (LSO) 策略场景**

   * **场景说明**：某些事务可能较大，未提交期间可能阻塞后续已提交消息的可读性；消费端不应看到尚未提交的事务消息。
   * **Kafka 角色**：客户端采用 `read_committed` 模式，Broker 基于 LSO 策略只返回 ≤ LSO 的消息；LSO 定义为第一个未完成事务的起始位置，过滤掉尚未提交的消息。已提交的但位于 LSO 之后的短事务消息，因被长事务阻塞，也暂时无法返回；这样做兼顾正确性与性能/内存压力。此外，Broker 维护 `.txnindex` 记录已回滚事务的区间，以便消费端在拉取时提前过滤回滚数据，减轻客户端缓存压力。

8. **事务超时与清理场景**

   * **场景说明**：若生产者开启事务后长期无进展（超时），或事务 ID 长期未使用，需回滚并回收资源。
   * **Kafka 角色**：

     * **事务操作超时**：客户端设定 transaction.timeout.ms，若超时未提交，协调者主动回滚事务；服务端有 max.transaction.timeout.ms 上限。
     * **事务 ID 过期**：协调者定期检查 transaction.id.expiration.ms，若某事务 ID 长期无请求，则清理对应元数据，释放 PID 等资源。

---

## 2. 关键设计及其理念、解决问题和作用

下面按逻辑模块或角色逐一列举，并说明设计初衷、解决的问题及在 Exactly-Once 流场景中的作用。

1. **Producer ID (PID) 与分区内序列号 + 幂等生产者**

   * **设计理念**：通过给每个生产者会话分配唯一 PID，并在每个 Topic-Partition 维度维护序列号，服务端只需保存有限最近批次信息即可判重与乱序，而非保存所有消息 ID。
   * **解决问题**：解决单会话中由于网络 ACK 丢失或重试导致的重复写入与乱序写入问题，实现幂等性（Exactly-Once at least on single-session, single-partition）。
   * **作用**：为后续跨会话幂等与事务提供基础；在每个 ProduceRequest 中携带 PID + epoch +序列号，Broker 检测重复或乱序并拒绝或忽略，从而保证单会话单分区写入的幂等性。

2. **Transactional ID + Epoch (Fencing) 机制**

   * **设计理念**：用户为流处理应用提供一个稳定的 Transactional ID；服务端为该 ID 分配 PID，并在每次初始化或重启时通过 epoch 隔离过期会话，避免“僵尸生产者”混入。
   * **解决问题**：跨会话恢复时，若旧会话崩溃，新会话仍能以同一 Transactional ID 继续或回滚上次未完成事务；防止多个生产者同时用同一 Transactional ID 导致竞态（fencing）。
   * **作用**：实现跨会话幂等写入：旧事务若未完成会被主动回滚；新会话获得更高 epoch，旧会话请求因 epoch 较低将被拒绝，从而保证只有当前活跃会话能提交。

3. **事务协调者 (Transaction Coordinator)**

   * **设计理念**：集中管理事务元数据（Transactional ID、PID、epoch、涉及分区列表、消费位点信息等），并在事务完成后统一负责广播提交/回滚控制消息。
   * **解决问题**：在多分区写入、多个操作原子性需求下，避免客户端对每个分区 Leader 逐一发送“提交/回滚”命令；也便于在协调者故障切换时通过集中日志恢复。
   * **作用**：

     * **分配与管理 PID**（与 ProducerIdManager 交互，从 ZooKeeper 或内部机制获取 ID 段，并维护最新状态）。
     * **跟踪事务涉及的所有 Topic-Partition**：在发送事务消息前，客户端通过 AddPartitionsToTxnRequest 通知协调者，协调者持久化到 \_\_transaction\_state。
     * **跟踪消费位点提交信息**：通过 AddOffsetsToTxnRequest 同步要提交的 \_\_consumer\_offsets 分区信息到事务元数据，并最终向 Group Coordinator 提交。
     * **提交/回滚决策**：接收 EndTxnRequest 后，更新 **transaction\_state 到 PREPARE*\* 状态，再异步向各分区 Leader 发送 WriteTxnMarker 控制消息；待所有控制消息成功持久化，再更新状态到 COMPLETE*\* 并释放资源。
     * **HA 与持久化**：利用内部 \_\_transaction\_state compacted Topic，多副本保证日志不丢失；故障切换后可从日志恢复正在进行的事务状态。
     * **定期回滚超时事务**、清理过期事务 ID。
     * **在故障场景中接管**：若客户端崩溃但 EndTxnRequest 已到达协调者，协调者可继续完成提交/回滚流程；若 EndTxnRequest 未到达，则超时自动回滚。

4. **内部事务状态 Topic (\_\_transaction\_state)**

   * **设计理念**：持久化事务元数据（Transactional ID → 当前状态、PID+epoch、涉及分区集合、提交位点信息等），类似一个分布式 KV 存储。
   * **解决问题**：提供 HA 恢复能力：协调者故障后可从该 Topic 读取最新状态；保证事务状态机在集群重启或切换时不丢失。
   * **作用**：记录事务状态转移（EMPTY → ONGOING → PREPARE\_COMMIT/ABORT → COMPLETE\_COMMIT/ABORT → DEAD），关键节点持久化；协调者通过该日志驱动后续控制消息发送与资源清理。

5. **控制消息 (Transaction Markers, WriteTxnMarkerRequest)**

   * **设计理念**：将“提交”或“回滚”决策以控制消息形式写入普通日志，与业务消息共存；消费端读取到控制消息后，得知对应事务消息是否可见。
   * **解决问题**：一方面避免在 Broker 内存中临时缓存大量未完成消息；另一方面让消费者通过读取日志中的控制消息来判断事务边界。
   * **作用**：事务协调者在 PREPARE 阶段向各分区 Leader 发送 WriteTxnMarker，Leader 将此写入日志。消费者在 read\_committed 模式下，见到控制消息后再决定允许看到之前的消息或忽略（依据提交或回滚）。协调者会不断重试直到所有分区成功写入控制消息，保证原子提交/回滚。

6. **ProducerIdManager / PID Snapshot 机制**

   * **设计理念**：服务端维护分配给各 Transactional ID 的 PID 段，并在内存中跟踪当前活跃 PID → 分区内最高序列号映射；通过定期快照减少重启恢复时读取全量日志的开销。
   * **解决问题**：避免每次 Broker 重启时扫描全部日志以重建 PID 序列号映射；提升恢复速度；同时确保在故障恢复后依然能进行幂等判重。
   * **作用**：在 Broker 启动或 Leader 切换时，通过 PID Snapshot 和日志中记录恢复 ProducerState；在运行时对 ProduceRequest 进行序列号校验；支持事务中对重复批次识别。

7. **Group Coordinator / \_\_consumer\_offsets 与 sendOffsetsToTransaction 设计**

   * **设计理念**：在事务中，消费者提交 offset 需与输出消息一同原子提交；借助 Producer 作为客户端在事务上下文中先向事务协调者同步将要提交的 offset 分区，再正式向 Group Coordinator 提交但不更新缓存，待事务提交后才让消费者看到新位点。
   * **解决问题**：避免先提交位点而输出消息回滚时出现“丢数据”或先输出消息而位点未提交时崩溃导致“重复处理”。
   * **作用**：

     * 生产者调用 sendOffsetsToTransaction()，先发 AddOffsetsToTxnRequest 给事务协调者，协调者将该 offset 分区信息加入事务元数据；收到响应后，再发 TxnOffsetCommitRequest 给 Group Coordinator，将 offset 写入 \_\_consumer\_offsets 日志但不更新缓存；只有当事务提交完成，协调者才能将该位点对消费者可见。
     * Group Coordinator 接收包含 PID/epoch 的 TxnOffsetCommitRequest，持久化 offset；但直到事务完成，不会更新消费进度缓存，保证原子性。

8. **Last Stable Offset (LSO) 与 read\_committed 消费模式**

   * **设计理念**：在消费端只返回“已提交且稳定”的消息，屏蔽尚未提交的事务消息；用 LSO 标记首个未完成事务的起始位置。
   * **解决问题**：避免消费者看到未完成或回滚事务的数据；同时避免服务器或客户端因缓存所有未提交消息导致 OOM 或性能下降。
   * **作用**：Broker 在 read\_committed 模式下根据分区 LSO，只返回 ≤ LSO 的消息；当长事务阻塞后续短事务，短事务即使已提交，也暂时不被返回，但可在后续重新计算 LSO 后消费；结合 .txnindex 提前过滤回滚事务区间，减轻客户端处理压力。

9. **.txnindex 回滚事务索引**

   * **设计理念**：Broker 为每个分区维护已回滚事务对应的日志起止位点索引，并持久化于 .txnindex 文件。
   * **解决问题**：让消费端在拉取时可以提前获知哪些区间的数据属于已回滚事务，从而跳过，无需客户端缓存后续等待控制消息。
   * **作用**：提高消费效率，减少因等待控制消息而占用内存或重复拉取的开销。

10. **事务超时与过期清理**

    * **设计理念**：设置事务超时时间（transaction.timeout.ms）以限制客户端完成事务的最长时限；协调者有最大可接受超时 max.transaction.timeout.ms；此外通过 transaction.id.expiration.ms 清理长期不活跃的事务 ID。
    * **解决问题**：避免事务无限期挂起占用资源；确保故障恢复后能及时回滚或释放；释放 PID 资源，防止元数据膨胀。
    * **作用**：协调者定期扫描并回滚超时事务；清理过期事务 ID & PID 映射，保证系统健康和资源回收。

11. **内部 HA 与故障恢复设计**

    * **设计理念**：利用 Kafka 本身多副本、Leader 选举机制，以及内部 compacted Topics（如 \_\_transaction\_state、\_\_consumer\_offsets），保证在 Broker 或协调者故障后可快速恢复状态。
    * **解决问题**：在分布式环境中，任何节点崩溃都不应导致事务状态不一致或丢失，保证 Exactly-Once 语义在故障场景下依然成立。
    * **作用**：

      * 协调者失败后，新任协调者读取 \_\_transaction\_state 恢复正在进行的事务状态并继续发送控制消息或回滚。
      * 分区 Leader 失败后，通过日志与 PID Snapshot 恢复 ProducerState；协调者重试 WriteTxnMarker；消费者继续消费已提交数据。
      * Group Coordinator 维护 \_\_consumer\_offsets 多副本以恢复 offset 提交。

12. **客户端状态机管理**

    * **设计理念**：在 Producer 客户端内部维护一组状态（UNINITIALIZED → INITIALIZING → READY → IN\_TRANSACTION → COMMITTING/ABORTING\_TRANSACTION → ...），确保在不同阶段只有合法操作，并在异常时进入 ABORTABLE\_ERROR 或 FATAL\_ERROR，触发回滚或关闭。
    * **解决问题**：避免客户端在不合适时机调用事务 API；在出错时能及时回滚并清理本地状态；配合服务端状态一致性。
    * **作用**：在调用 initTransactions、beginTransaction、send/sendOffsetsToTransaction、commitTransaction/abortTransaction 各阶段，通过状态转换与校验保证正确调用顺序；在异常或超时场景自动回滚或报告错误。

13. **消费模式区分（read\_uncommitted vs read\_committed）**

    * **设计理念**：提供两种消费语义：默认 read\_uncommitted（可见所有消息，包括未提交事务消息）；read\_committed（仅见已提交且稳定的）。
    * **解决问题**：支持不同应用需求：某些场景可能容忍看到未提交消息（如内部分析）；多数场景需要 Exactly-Once 语义，则使用 read\_committed，屏蔽未提交或回滚数据。
    * **作用**：在客户端配置中设置 isolation.level; Broker 根据 LSO 及控制消息决定返回哪些消息。

14. **生产者与协调者的交互协议设计（各种 Request 类型）**

    * **设计理念**：通过一系列 Request/Response：FindCoordinatorRequest、InitProducerIdRequest、AddPartitionsToTxnRequest、ProduceRequest（携带 PID+序列号+事务标识）、AddOffsetsToTxnRequest、TxnOffsetCommitRequest、EndTxnRequest、WriteTxnMarkerRequest（由协调者发起）、确保每一步都有明确目的、可持久化与可重试。
    * **解决问题**：结构化定义各阶段交互，使客户端和服务端对事务元数据更新、消息写入、位点提交等都有清晰流程；支持重试与故障恢复；避免协议混乱。
    * **作用**：各请求携带必要标识（Transactional ID、PID、epoch、分区列表、offset、commit/abort 标志等），服务端对应处理并持久化，协调者集中管理后续步骤。

15. **ZooKeeper 或内部机制管理 PID 段分配**

    * **设计理念**：为产生全局唯一的 PID，协调者通过 ProducerIdManager 从 ZooKeeper（或新版 Kafka 可内置替代）申请 ID 段；每次申请一段（如 1000 IDs），减少频繁交互。
    * **解决问题**：确保 PID 唯一；高效分配；支持扩展与 HA。
    * **作用**：协调者在 InitPidRequest 阶段分配 PID，并记录最新分配区间；客户端获得 PID 后用于后续幂等与事务；Broker 重启恢复时也需恢复 PID-序列号映射。

16. **事务 Marker 发送与重试机制**

    * **设计理念**：协调者在 PREPARE 阶段发出 WriteTxnMarkerRequest，并不断重试直至所有分区确认成功后，才更新状态为 COMPLETE；任何失败导致无限重试或故障切换后继续重试。
    * **解决问题**：保证事务消息的提交或回滚通知可靠送达，即使在网络或 Broker 故障时亦可最终一致；避免半提交状态导致数据可见性错误。
    * **作用**：协调者内部维护待发送队列（markersQueuePerBroker），对失败分区重新 enqueue；整体成功后触发事务完成清理；若协调者自身失败，新协调者从 \_\_transaction\_state 恢复并继续发送。

17. **事务状态机与状态转移**

    * **设计理念**：在服务端与客户端分别维护明确状态机，描述生命周期；状态转换时伴随持久化或必要的交互。
    * **解决问题**：确保仅在合法阶段执行对应操作；在异常或故障时能根据当前状态采取回滚、重试或清理；便于理解和维护。
    * **作用**：

      * 服务端状态：EMPTY → ONGOING → PREPARE\_COMMIT / PREPARE\_ABORT → COMPLETE\_COMMIT / COMPLETE\_ABORT → DEAD → (清理)；协调者根据 EndTxnRequest、超时检测等进行状态转移并持久化。
      * 客户端状态：UNINITIALIZED → INITIALIZING → READY → IN\_TRANSACTION → COMMITTING\_TRANSACTION / ABORTING\_TRANSACTION → READY 或 ABORTABLE\_ERROR / FATAL\_ERROR；保证 API 调用顺序及异常处理。

---

> **总结**：Kafka 在流处理场景下端到端 Exactly-Once 能力，核心依赖于幂等生产者（PID+序列号+判重）、事务机制（Transactional ID+Epoch+Transaction Coordinator+内部 \_\_transaction\_state 日志+控制消息+LSO/read\_committed+.txnindex）、以及消费端配合（sendOffsetsToTransaction、read\_committed）。各设计模块相互协作，通过分层协议与持久化机制，在分布式、故障、网络抖动场景下仍能提供正确的恰好一次处理语义。
