---
创建日期: 2025-04-12
---

### 它有什么用？

`Condition` 是 Java 并发包（java.util.concurrent）中提供的一种高级线程协作机制，它是对 `Object` 类中 `wait()`、`notify()` 和 `notifyAll()` 方法的增强替代，提供了更加强大且灵活的功能。相比传统机制，`Condition` 支持创建多个条件队列，从而实现更细粒度、更精确的线程等待与唤醒控制，有效解决了 `wait/notify` 随机唤醒带来的性能问题。

**没有 Condition 会遇到什么麻烦？**

在使用 `wait()` 和 `notify()` 时，会遇到一些严重的问题，尤其是在复杂的并发场景中：所有线程都共享同一个等待队列，如果存在多个条件（例如“队列不满”和“队列不空”），就无法区分应该唤醒哪一类线程。`notify()` 只能随机唤醒一个线程，而 `notifyAll()` 又可能唤醒所有线程，导致大量无效唤醒，从而严重影响程序的执行效率。

当消费者消费完一个数据后调用 `notify()`，此时可能唤醒的是等待中的生产者线程，也可能是另一个正在等待消费的消费者线程（如果存在）。理想情况下，应唤醒生产者线程以继续生产。但如果唤醒的是另一个消费者，而缓冲区中此时没有新数据，该线程将立刻再次进入等待状态，造成不必要的上下文切换和性能开销。

**Condition 解决了什么问题？**

`Condition` 接口通常与 `Lock` 搭配使用，一个 `Lock` 可以创建多个 `Condition` 对象。每个 `Condition` 表示一个明确的等待条件，并对应一个独立的等待队列，用于存放因某个特定条件未满足而被挂起的线程。这样可以将不同的条件分开管理，实现多个等待队列的精细控制，从而在唤醒线程时更具针对性，避免无效唤醒。

可以将 `Condition` 类比为餐厅中的不同等候区域：每个等候区（即一个 `Condition` 对象）负责管理因不同原因而等待的顾客。

例如，餐厅可能设有多个等候区：一部分顾客在等待座位清理完成，另一部分顾客则在等待有空桌可用。根据等待的具体原因，顾客会被分别引导到对应的等候区域。

当顾客到达餐厅时，如果座位尚未准备好，他们会被安排在特定的等候区等待（调用 `await()`）。一旦条件满足（如座位准备完毕或有空桌出现），服务员就会通知对应等候区的顾客可以入座（调用 `signal()` 或 `signalAll()`）。

在底层实现中，一个 `Lock` 对象可以关联多个 `Condition`，每个 `Condition` 都维护一个独立的等待队列。线程通过调用 `await()` 方法进入相应的等待队列并进入阻塞状态，直到被对应的 `signal()` 或 `signalAll()` 唤醒并重新竞争锁资源。

### 常用的方法

**创建实现类**

`Condition` 是一个接口，通常不直接实现，而是通过 `ReentrantLock` 来创建：

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition(); // 创建 Condition 实例
```

**等待方法**

```java
void await() throws InterruptedException // 使当前线程进入等待状态并释放锁。线程会一直等待，直到被其他线程通过 signal() 或 signalAll() 唤醒，或者等待被中断。
boolean await(long time, TimeUnit unit) throws InterruptedException // 最多等待指定时间，被唤醒返回 true，超时返回 false
```

**唤醒方法**

```java
void signal() // 唤醒一个在此 Condition 上等待的线程（谁被唤醒是非确定的）

void signalAll() // 唤醒所有在此 Condition 上等待的线程
```
