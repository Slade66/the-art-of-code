---
创建日期: 2025-04-10
---

发音：塞么佛。

`Semaphore`（信号量）是一种计数锁，用于控制同时访问某一资源的线程数量，常用于“限制并发数”。它通过许可机制实现限流：线程在进入临界区之前必须先获取许可，任务完成后再释放许可；如果当前没有可用的许可，线程就会被阻塞，直到有线程释放许可为止。

```java
public class SemaphoreDemo {
    public static void main(String[] args) {
        int permits = 3; // 只允许3个线程同时执行
        Semaphore semaphore = new Semaphore(permits);

        for (int i = 0; i < 10; i++) {
            final int threadNum = i;
            new Thread(() -> {
                try {
                    semaphore.acquire(); // 获取许可证（坑位）
                    System.out.println("线程 " + threadNum + " 获取许可，开始执行任务");
                    Thread.sleep(1000); // 模拟任务执行
                    System.out.println("线程 " + threadNum + " 任务执行完毕，释放许可");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release(); // 释放许可证
                }
            }).start();
        }
    }
}
```

`synchronized` 和 `ReentrantLock` 都是独占锁，同一时刻只允许一个线程访问临界区。而 `Semaphore` 可以同时允许 N 个线程并发访问资源，用于控制并发数量。`CountDownLatch` 用于让一个或多个线程等待其它线程完成任务后再继续执行。`CyclicBarrier` 则是让多个线程在到达某个屏障点后统一触发下一步操作，适合阶段性同步。
