---
创建日期: 2025-04-07
---

## 常用方法

### 线程协作

这三个方法必须在持有锁的前提下调用（即位于 `synchronized` 块内部），否则会抛出 `IllegalMonitorStateException` 异常。

在 JVM 中，每个对象在需要加锁时会关联一个 `ObjectMonitor`（监视器锁）。其中包含两个关键的队列：
`_entry_list`：表示正在尝试进入 `synchronized` 块但尚未获得锁的线程（处于阻塞状态）。
`_wait_set`：表示已经调用了 `wait()` 方法、主动释放锁、等待被唤醒的线程。

线程在调用 `wait()` 方法后会释放持有的锁并进入等待状态；而 `notify()` 或 `notifyAll()` 只是负责唤醒等待中的线程，并不会释放当前线程持有的锁。

#### wait()

使当前线程挂起并释放锁，并加入对象的等待队列，直到被其它线程 `notify()` 或 `notifyAll()` 唤醒。

最终会调用 JVM 内部的 native 方法 `ObjectMonitor::wait()`。

#### notify()

唤醒一个在该对象上调用 `wait()` 而挂起的线程。

`notify()` 只是将一个等待线程从 Wait Set 移入 Entry List，使其进入“就绪状态”。但被唤醒的线程不会立刻执行，它必须等当前线程释放锁后，才能参与锁的竞争，并在成功获取锁后继续运行。

在 `notify()` 后的 `wait()` 不会被唤醒。

#### notifyAll()

唤醒所有在该对象上调用 `wait()` 而挂起的线程。

`notifyAll()` 会将 Wait Set 中的所有线程移动到 Entry List。它们随后会尝试重新获取锁，但仍需要等待当前线程释放锁后，才能逐个恢复执行。
