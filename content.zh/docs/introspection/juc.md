---
title: Java.util.concurrent 包
---

Java 中的 **JUC（java.util.concurrent）** 是并发编程的核心包，提供了大量强大、高性能的工具类来简化线程操作和并发控制。

下面是对 JUC 中**最常用类和接口的详细分类与解析**，帮助你构建对 Java 并发编程的完整认识。

---

## 🌳 一、JUC 核心模块总览

```text
java.util.concurrent.*
├── Executor 框架（线程池）
├── 同步器（Locks、CountDownLatch、Semaphore、CyclicBarrier）
├── 并发集合（ConcurrentHashMap、CopyOnWriteArrayList 等）
├── 原子变量（AtomicXXX）
├── Fork/Join 框架
├── CompletableFuture、Future
├── ScheduledExecutor、BlockingQueue 等
```

---

## 🚀 二、Executor 框架（线程池相关）

| 类/接口                       | 说明                                         |
| -------------------------- | ------------------------------------------ |
| `Executor`                 | 最基础的接口，只有一个方法 `execute(Runnable)`          |
| `ExecutorService`          | 提供 `submit()`、`shutdown()` 等高级控制           |
| `ThreadPoolExecutor`       | 可自定义线程池参数的核心实现类                            |
| `ScheduledExecutorService` | 支持定时或周期任务执行                                |
| `Executors`                | 工厂类，提供常用线程池的创建方法（如 `newFixedThreadPool()`） |

### ☑️ 推荐使用方式

避免 `Executors.newXxx()` 静态方法，建议自己构造 `ThreadPoolExecutor`：

```java
ExecutorService pool = new ThreadPoolExecutor(
    4, 8, 60, TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(100)
);
```

---

## 🧱 三、同步器（Lock、信号量、栅栏等）

### 1. Lock/Condition

| 类/接口            | 说明                                 |
| --------------- | ---------------------------------- |
| `Lock`          | 比 `synchronized` 更灵活，支持中断、非阻塞获取    |
| `ReentrantLock` | 可重入锁，支持公平/非公平锁                     |
| `ReadWriteLock` | 读写分离，提高并发性                         |
| `Condition`     | 类似 `Object.wait/notify`，配合 Lock 使用 |

示例：

```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区
} finally {
    lock.unlock();
}
```

---

### 2. 线程协作工具类

| 类                | 说明                |
| ---------------- | ----------------- |
| `CountDownLatch` | 一个或多个线程等待其他线程完成   |
| `CyclicBarrier`  | 多个线程互相等待，到达屏障点后继续 |
| `Semaphore`      | 控制并发访问的许可证（限流）    |
| `Exchanger`      | 两个线程数据交换          |

示例：`CountDownLatch`

```java
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // do work
        latch.countDown();
    }).start();
}
latch.await(); // 主线程等待子线程完成
```

---

## 💥 四、原子类（AtomicXXX）

| 类                               | 说明                     |
| ------------------------------- | ---------------------- |
| `AtomicInteger/AtomicLong`      | 原子整型操作（如递增）            |
| `AtomicReference`               | 原子更新对象引用               |
| `AtomicStampedReference`        | 防止 ABA 问题的引用类          |
| `LongAdder` / `LongAccumulator` | 高并发下比 `AtomicLong` 更高效 |

示例：

```java
AtomicInteger count = new AtomicInteger();
count.incrementAndGet();  // 原子 +1
```

---

## 🔄 五、并发集合类

| 类                       | 说明                                                   |
| ----------------------- | ---------------------------------------------------- |
| `ConcurrentHashMap`     | 支持高并发读写的 Map（JDK8 以后性能大幅提升）                          |
| `CopyOnWriteArrayList`  | 写时复制，读多写少场景                                          |
| `ConcurrentLinkedQueue` | 无锁队列                                                 |
| `BlockingQueue`（接口）     | 支持阻塞的队列，如 `ArrayBlockingQueue`、`LinkedBlockingQueue` |

示例：

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key", 1);
```

---

## ⚙️ 六、阻塞队列（用于生产者-消费者）

| 类                     | 特性               |
| --------------------- | ---------------- |
| `ArrayBlockingQueue`  | 有界，数组结构，适合固定容量   |
| `LinkedBlockingQueue` | 无界或有界，链表结构       |
| `DelayQueue`          | 带延迟元素的队列         |
| `SynchronousQueue`    | 不存储元素的队列（用于线程交接） |

常用于线程池或生产者消费者模型中。

---

## 🧠 七、Future、CompletableFuture、ForkJoin

### `Future` & `Callable`

* 提交任务后返回 Future，可通过 `get()` 阻塞获取结果。

```java
Future<Integer> f = pool.submit(() -> 1 + 1);
System.out.println(f.get());
```

### `CompletableFuture`

* 支持链式异步任务、异常处理、组合计算（JDK 8+）

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenAccept(System.out::println);
```

### `ForkJoinPool`

* 分而治之的并行计算框架，适合大规模数据递归处理

---

## 📌 总结结构图（经典分类）

```
java.util.concurrent
├── Executors / ThreadPoolExecutor         ← 线程池
├── Locks (Lock, ReadWriteLock)           ← 手动加锁
├── Synchronizers (CountDownLatch, ...)   ← 协作
├── Atomic variables (AtomicXXX)          ← 原子类
├── Concurrent collections                ← 高并发容器
├── BlockingQueue / DelayQueue            ← 阻塞队列
├── Future / CompletableFuture            ← 异步结果
├── ForkJoinPool                          ← 并行计算框架
```

---

## 📘 推荐学习顺序

1. `ExecutorService` / `ThreadPoolExecutor` ← 打基础
2. `ReentrantLock` / `CountDownLatch` / `Semaphore` ← 并发控制工具
3. `ConcurrentHashMap` / `CopyOnWriteArrayList` ← 集合并发化
4. `AtomicInteger` / `LongAdder` ← 原子类
5. `CompletableFuture` ← 现代异步方式
6. `ForkJoinPool` ← 进阶任务分治模型

---