# 目录
- [队列](#队列)
  - [非阻塞队列（适用于单线程或手动同步）](#非阻塞队列适用于单线程或手动同步)
  - [阻塞队列（`java.util.concurrent` 包）](#阻塞队列javautilconcurrent-包)
  - [无锁/高性能队列（非阻塞并发队列）](#无锁高性能队列非阻塞并发队列)
  - [对比总结](#对比总结)
  - [典型使用场景](#典型使用场景)
- [LinkedList 的主要特性](#linkedlist-的主要特性)
  - [List 接口的方法](#list-接口的方法)
  - [Deque（双端队列）方法](#deque双端队列方法)
  - [Queue 方法](#queue-方法)
- [创建线程的两种方式](#创建线程的两种方式)
  - [继承 `Thread` 类](#继承-thread-类)
  - [实现 `Runnable` 接口（推荐）](#实现-runnable-接口推荐)
  - [`Thread` 常用方法一览](#thread-常用方法一览)

# 队列
## 非阻塞队列（适用于单线程或手动同步）
| 实现类             | 特点                                    |
| --------------- | ------------------------------------- |
| `LinkedList`    | 实现了 `Deque` 和 `Queue`，可以作为普通队列或双端队列使用 |
| `PriorityQueue` | 元素按优先级排列（非 FIFO），不支持并发                |
| `ArrayDeque`    | 高性能双端队列，非线程安全                         |
## 阻塞队列（`java.util.concurrent` 包）
| 实现类                     | 特点                                   |
| ----------------------- | ------------------------------------ |
| `ArrayBlockingQueue`    | 有界阻塞队列，数组实现，支持 FIFO                  |
| `LinkedBlockingQueue`   | 可选容量的阻塞队列，链表实现，FIFO                  |
| `PriorityBlockingQueue` | 无界，带优先级排序，不保证 FIFO                   |
| `DelayQueue`            | 元素按延迟时间排序，仅在到期后才能取出                  |
| `SynchronousQueue`      | 每个插入操作必须等待一个对应的取出操作（零容量）             |
| `LinkedTransferQueue`   | 支持生产者等待消费者、容量无界                      |
| `BlockingDeque`         | 支持阻塞的双端队列操作（如 `LinkedBlockingDeque`） |
## 无锁/高性能队列（非阻塞并发队列）
| 实现类                     | 特点                  |
| ----------------------- | ------------------- |
| `ConcurrentLinkedQueue` | 无界、基于链表、适用于高并发（非阻塞） |
| `ConcurrentLinkedDeque` | 双端无锁队列，适合并发环境       |
## 对比总结
| 队列类型                    | 是否阻塞 | 是否线程安全 | 是否有界   | 是否支持优先级 |
| ----------------------- | ---- | ------ | ------ | ------- |
| `LinkedList`            | 否    | 否      | 否      | 否       |
| `PriorityQueue`         | 否    | 否      | 否      | 是       |
| `ArrayBlockingQueue`    | 是    | 是      | 是      | 否       |
| `LinkedBlockingQueue`   | 是    | 是      | 可选     | 否       |
| `PriorityBlockingQueue` | 是    | 是      | 否      | 是       |
| `DelayQueue`            | 是    | 是      | 否      | 是（延迟时间） |
| `SynchronousQueue`      | 是    | 是      | 是（零容量） | 否       |
| `ConcurrentLinkedQueue` | 否    | 是      | 否      | 否       |
## 典型使用场景
任务调度 / 消息中转： `LinkedBlockingQueue`, `DelayQueue`

高并发日志采集： `ConcurrentLinkedQueue`

线程池任务队列： `ArrayBlockingQueue`, `SynchronousQueue`

定时/优先级任务： `PriorityBlockingQueue`, `DelayQueue`
# LinkedList 的主要特性
双向链表实现：插入、删除元素效率高（相较于 ArrayList）

元素可重复，允许 null

非线程安全（需要手动同步）
## List 接口的方法
| 方法                          | 说明          |
| --------------------------- | ----------- |
| `add(E e)`                  | 添加元素到末尾     |
| `add(int index, E element)` | 指定位置插入元素    |
| `remove(int index)`         | 移除指定位置的元素   |
| `remove(Object o)`          | 删除第一个匹配的元素  |
| `get(int index)`            | 获取指定位置的元素   |
| `set(int index, E element)` | 设置指定位置的元素   |
| `indexOf(Object o)`         | 查找元素首次出现的位置 |
| `lastIndexOf(Object o)`     | 查找元素最后出现的位置 |
| `clear()`                   | 清空所有元素      |
| `size()`                    | 获取元素数量      |
| `isEmpty()`                 | 判断是否为空      |
## Deque（双端队列）方法
| 方法                | 说明                      |
| ----------------- | ----------------------- |
| `addFirst(E e)`   | 头部添加元素                  |
| `addLast(E e)`    | 尾部添加元素                  |
| `removeFirst()`   | 移除并返回第一个元素              |
| `removeLast()`    | 移除并返回最后一个元素             |
| `getFirst()`      | 获取第一个元素，不移除             |
| `getLast()`       | 获取最后一个元素，不移除            |
| `offerFirst(E e)` | 头部插入元素，失败返回 false       |
| `offerLast(E e)`  | 尾部插入元素，失败返回 false       |
| `pollFirst()`     | 获取并移除第一个元素，队列空时返回 null  |
| `pollLast()`      | 获取并移除最后一个元素，队列空时返回 null |
| `peekFirst()`     | 查看第一个元素，不移除             |
| `peekLast()`      | 查看最后一个元素，不移除            |
## Queue 方法
| 方法           | 说明         |
| ------------ | ---------- |
| `offer(E e)` | 添加元素到队尾    |
| `poll()`     | 取出并移除队头元素  |
| `peek()`     | 查看队头元素但不移除 |

# 创建线程的两种方式
## 继承 `Thread` 类
```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread is running");
    }
}

MyThread t = new MyThread();
t.start();  // 启动线程
```
## 实现 `Runnable` 接口（推荐）
```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Thread is running");
    }
}

Thread t = new Thread(new MyRunnable());
t.start();
```
## `Thread` 常用方法一览
| 方法名                            | 说明                       |
| ------------------------------ | ------------------------ |
| `start()`                      | 启动线程（会调用 `run()` 方法）     |
| `run()`                        | 线程执行的任务内容（可重写）           |
| `sleep(long millis)`           | 当前线程睡眠指定时间（毫秒）           |
| `join()`                       | 等待某个线程执行完                |
| `interrupt()`                  | 中断线程（并不会强制停止）            |
| `isInterrupted()`              | 检查线程是否被中断                |
| `setPriority(int newPriority)` | 设置线程优先级（1\~10）           |
| `getPriority()`                | 获取线程优先级                  |
| `setName(String name)`         | 设置线程名                    |
| `getName()`                    | 获取线程名                    |
| `setDaemon(boolean on)`        | 设置为守护线程（在 `start()` 前调用） |
| `isDaemon()`                   | 判断是否是守护线程                |
| `currentThread()`              | 获取当前正在执行的线程对象（静态方法）      |
| `yield()`                      | 当前线程让出 CPU 执行权（不一定成功）    |
| `isAlive()`                    | 判断线程是否仍在运行中              |
