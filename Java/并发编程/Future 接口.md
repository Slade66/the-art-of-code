---
创建日期: 2025-04-11
---

`Future` 用于表示一个异步执行任务的结果。它解决了这样的问题：多线程任务执行完毕后，如何获取计算结果、检查任务状态，或进行取消和超时控制。

`Future` 通常用于异步编程，可避免程序在执行耗时任务时陷入原地等待。当需要处理一个耗时操作时，我们可以将其交给子线程异步执行，主线程则继续处理其他任务，无需傻等。待需要结果时，再通过 `Future` 获取该任务的执行结果。

在编写异步代码和处理任务结果时，几乎不可避免地会接触到 `Future` 或其衍生的更高级接口（如 `CompletableFuture`）。

常用方法：
- `get()`，阻塞获取任务结果
- `get(timeout, unit)`，同上，但设置了超时时间
- `cancel(true/false)`，取消任务
- `isDone()`，检查任务是否完成
- `isCancelled()`，检查任务是否被取消

线程池的 `submit()` 方法用于提交一个带有返回值的任务，并返回一个表示该任务结果的 `Future` 对象：

```java
public class FutureExample {
    public static void main(String[] args) throws Exception {
        // ① 创建线程池
        ExecutorService executor = Executors.newSingleThreadExecutor();

        // ② 提交异步任务（必须用 Callable，才能返回结果）
        Future<Integer> future = executor.submit(() -> {
            Thread.sleep(1000); // 模拟耗时
            return 1 + 1;
        });

        System.out.println("主线程继续干别的...");

        // ③ 获取结果（会阻塞，直到任务完成）
        Integer result = future.get();

        System.out.println("任务结果是：" + result);

        // ④ 关闭线程池
        executor.shutdown();
    }
}
```
