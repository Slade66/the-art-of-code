---
创建日期: 2025-04-10
---

它是一个倒计时锁，可以让一个或多个线程等待其它线程完成任务之后再继续执行。它解决的是线程之间的协作和等待问题，比如主线程要等多个子线程都执行完，才能继续往下走，这时候就可以用它。

在初始化时通过 `new CountDownLatch(int count)` 指定计数次数。调用 `await()` 方法的线程会被阻塞，直到其它线程通过调用 `countDown()` 将计数器减到 0，阻塞的线程才会继续执行。

```java
public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        int taskCount = 3;
        CountDownLatch latch = new CountDownLatch(taskCount); // 初始化计数器为3

        for (int i = 0; i < taskCount; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " 开始执行任务...");
                try {
                    Thread.sleep(1000); // 固定耗时1秒
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 执行完毕！");
                latch.countDown(); // 每个线程完成后计数器减1
            }, "子线程-" + i).start();
        }

        System.out.println("主线程等待所有子线程完成任务...");
        latch.await(); // 主线程阻塞，直到计数器为0
        System.out.println("所有子线程已完成，主线程继续执行！");
    }
}
```

它是一次性的，计数器一旦减到 0 就无法重置，不能重复使用。

```java
public class CountDownLatchReuseDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(1);

        // 第一次使用
        Thread t1 = new Thread(() -> {
            System.out.println("线程1等待中...");
            try {
                latch.await();
                System.out.println("线程1继续执行");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t1.start();

        Thread.sleep(1000);
        System.out.println("主线程调用 countDown()");
        latch.countDown(); // 计数器变为 0，线程1被唤醒

        t1.join(); // 等待线程1执行完

        System.out.println("尝试再次使用同一个 CountDownLatch");

        // 第二次使用同一个 latch（不会再阻塞了）
        Thread t2 = new Thread(() -> {
            System.out.println("线程2等待中...");
            try {
                latch.await(); // 不会阻塞，直接通过
                System.out.println("线程2继续执行");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t2.start();
        t2.join();
    }
}
```
