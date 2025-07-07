---
title: Java 线程池
---

在 Java 中使用线程池（如 `ThreadPoolExecutor`）时，理解其参数非常关键，这些参数直接影响线程的**性能**、**可扩展性**和**稳定性**。下面详细介绍 `ThreadPoolExecutor` 构造函数的各个参数及其作用：

---

## 🧱 构造函数定义

```java
public ThreadPoolExecutor(
    int corePoolSize,
    int maximumPoolSize,
    long keepAliveTime,
    TimeUnit unit,
    BlockingQueue<Runnable> workQueue,
    ThreadFactory threadFactory,
    RejectedExecutionHandler handler
)
```

---

## 📌 参数详细说明

| 参数                | 类型                         | 说明                                                                                  |
| ----------------- | -------------------------- | ----------------------------------------------------------------------------------- |
| `corePoolSize`    | `int`                      | 核心线程数，线程池中**始终保留**的线程数，即使它们空闲也不会被销毁（除非设置了 `allowCoreThreadTimeOut(true)`）           |
| `maximumPoolSize` | `int`                      | 线程池中**允许的最大线程数**，当任务过多并且队列满了时才会创建超过 core 数量的线程                                      |
| `keepAliveTime`   | `long`                     | 非核心线程空闲多久会被销毁；如果设置了 `allowCoreThreadTimeOut(true)`，核心线程也会使用此超时                      |
| `unit`            | `TimeUnit`                 | 上述 keepAliveTime 的单位，例如 `TimeUnit.SECONDS`                                          |
| `workQueue`       | `BlockingQueue<Runnable>`  | 用来缓存等待执行的任务的队列，例如 `ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue` 等 |
| `threadFactory`   | `ThreadFactory`            | 自定义线程创建方式，例如设置线程名、是否为 daemon 等                                                      |
| `handler`         | `RejectedExecutionHandler` | 拒绝策略，当线程池和队列都满时用来处理新提交的任务                                                           |

---

## 🎯 工作机制简述

当一个任务提交到线程池时：

1. 当前线程数 < `corePoolSize` → 创建新线程处理任务；
2. 否则 → 放入 `workQueue`；
3. 如果队列满了 & 当前线程数 < `maximumPoolSize` → 再创建线程；
4. 如果线程已达最大值且队列也满了 → 执行 **拒绝策略 handler**。

---

## 🧪 队列类型的区别（workQueue）

| 类型                      | 特点                 | 适用场景                                            |
| ----------------------- | ------------------ | ----------------------------------------------- |
| `ArrayBlockingQueue`    | 有界队列，先进先出          | 推荐，控制最大任务数                                      |
| `LinkedBlockingQueue`   | 无界（或非常大）队列         | 容易导致线程数不增长，适合非突发流量                              |
| `SynchronousQueue`      | 不缓存任务，任务必须直接交给线程处理 | 适合高并发、低延迟场景，如 `Executors.newCachedThreadPool()` |
| `PriorityBlockingQueue` | 按优先级处理任务           | 需要排序执行任务的场景                                     |

---

## 🔁 拒绝策略（RejectedExecutionHandler）

当任务无法被处理时执行的回调策略，常用的有：

| 策略类                   | 行为                                 | 说明                |
| --------------------- | ---------------------------------- | ----------------- |
| `AbortPolicy`（默认）     | 抛出 `RejectedExecutionException` 异常 | 最安全，防止任务丢失        |
| `CallerRunsPolicy`    | 由提交任务的线程执行该任务                      | 可回退，降低任务提交速度      |
| `DiscardPolicy`       | 直接丢弃任务，无任何提示                       | 有丢任务风险            |
| `DiscardOldestPolicy` | 丢弃队列头部最旧任务，再尝试执行当前任务               | 适合实时任务，牺牲旧任务保障新任务 |

---

## 🛠 示例代码

```java
ExecutorService executor = new ThreadPoolExecutor(
    4,                    // corePoolSize
    8,                    // maximumPoolSize
    60L,                  // keepAliveTime
    TimeUnit.SECONDS,     // unit
    new ArrayBlockingQueue<>(100), // workQueue
    Executors.defaultThreadFactory(), // threadFactory
    new ThreadPoolExecutor.AbortPolicy() // handler
);
```

---

## 🧠 总结建议

| 建议项                 | 说明                       |
| ------------------- | ------------------------ |
| 明确设置核心和最大线程数        | 不要依赖默认线程池，避免 OOM 或任务堆积   |
| 使用有界队列              | 限制任务总数，防止内存溢出            |
| 合理选择拒绝策略            | 生产环境不要使用 `DiscardPolicy` |
| 自定义 ThreadFactory   | 可命名线程，方便排查问题             |
| 设置合理的 keepAliveTime | 防止线程资源浪费或频繁销毁创建          |