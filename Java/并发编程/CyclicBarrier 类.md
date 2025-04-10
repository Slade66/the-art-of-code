---
创建日期: 2025-04-10
---

`CyclicBarrier` 用于让一组线程在栅栏点（Barrier）处相互等待，只有当所有线程都到达屏障点后，才能统一继续执行后续任务。

```java
public class CyclicBarrierExample {
    public static void main(String[] args) {
        int threadCount = 3;

        // 创建一个 CyclicBarrier，等待 3 个线程，所有线程到齐后继续执行，并触发回调
        CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> {
            System.out.println("所有线程到齐，开始统一执行！");
        });

        // 启动3个线程，每个线程完成自己的前置逻辑后调用 await()
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                try {
                    System.out.println(Thread.currentThread().getName() + " 执行任务中...");
                    Thread.sleep(1000); // 模拟任务
                    System.out.println(Thread.currentThread().getName() + " 到达屏障，等待其他线程...");
                    barrier.await(); // 等待其它线程到达
                    System.out.println(Thread.currentThread().getName() + " 通过屏障，继续执行后续任务");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }, "线程-" + i).start();
        }
    }
}
```

支持重复使用。`CyclicBarrier` 就像一个可重置的闸门，一批线程到达之后统一放行，然后这个闸门会自动恢复，可以继续拦住下一批线程，再次等待“全部到齐”才继续。这个“循环使用”就是 cyclic 的含义。而 `CountDownLatch` 只能用一次，计数归 0 后不能再用了。

