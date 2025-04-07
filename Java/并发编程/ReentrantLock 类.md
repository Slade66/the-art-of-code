---
创建日期: 2025-04-06
---

### ReentrantLock 是什么？

`ReentrantLock` 是 Java 中 `java.util.concurrent.locks` 包下的一个可重入互斥锁，用于在并发环境中保护共享资源，防止被多个线程同时访问。它是对 `synchronized` 的一种显式替代，提供了更灵活且功能更强大的锁控制机制。

### ReentrantLock vs synchronized

相同点：两者都属于可重入锁（Reentrant Lock），同一个线程在持有同一把锁的情况下可以再次获得该锁而不会发生死锁。

不同点：
1. 实现方式：`synchronized` 是语言级关键字，由 JVM 实现；`ReentrantLock` 是 Java 类库提供的锁，基于 API 实现。
2. 加锁与释放：`synchronized` 隐式加锁、自动释放；`ReentrantLock` 需要手动调用 `lock()` 和 `unlock()`。
3. 公平性：`synchronized` 只能非公平；`ReentrantLock` 支持公平锁和非公平锁（通过构造函数指定）。
4. 可中断性：`synchronized` 不可中断，线程只能一直等待；`ReentrantLock` 支持中断（`tryLock(timeout)`、`lockInterruptibly()`）。
5. 锁优化：`synchronized` 支持锁升级（偏向锁、轻量级锁、重量级锁）；`ReentrantLock` 无锁升级机制。

如果对锁的控制粒度要求不高，使用 `synchronized` 即可，语法简单、易于维护。`ReentrantLock` 胜在功能丰富，适用于更复杂的业务场景。

### ReentrantLock 怎么用？

#### ReentrantLock()

创建一个非公平锁。

#### ReentrantLock(boolean fair)

传入 `true` 表示创建公平锁，传入 `false` 表示创建非公平锁（等同于使用无参构造方法 `ReentrantLock()`）。

#### lock()

获取锁，若锁已被占用则阻塞等待。

#### unlock()

释放锁。必须在 `finally` 中调用，确保始终释放锁，避免出异常无法释放。

#### tryLock()

尝试获取锁，立即返回 `true` 或 `false`，不会阻塞。

#### tryLock(long timeout, TimeUnit unit)

用于尝试在指定的超时时间内获取锁。如果在超时时间内成功获取锁，则返回 `true`；如果超时仍未获取到锁，则返回 `false`。

通过设置合理的等待时间，当线程在长时间内无法获取锁时，可以选择主动放弃，从而避免资源僵持和死锁，使线程在未能及时获取锁的情况下，能够继续执行其它替代操作。

#### lockInterruptibly()

与 `lock()` 类似，但可响应中断信号。
