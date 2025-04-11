---
创建日期: 2025-04-01
---

`CompletableFuture` 是 Java 8 引入的强大异步编程工具，它不仅实现了 `Future` 接口，对其功能进行了增强，还实现了 `CompletionStage` 接口，用于描述异步计算的各个阶段。通过 `CompletableFuture`，我们可以将复杂的计算过程拆分为多个阶段，并以流水线的形式进行组合与处理。它支持异步执行任务，不会阻塞主线程，并可在任务完成后自动触发回调函数来处理结果。

#### ForkJoinPool

`CompletableFuture` 默认使用 `ForkJoinPool.commonPool()` 作为线程池来执行异步任务，也可以传入自定义的线程池。

默认情况下，`ForkJoinPool` 的线程数通常设置为可用 CPU 核数。如果任务是 CPU 密集型，建议线程数与 CPU 核数接近；如果任务是 I/O 阻塞型，可以适当增大线程数。

它的名称来源于其任务处理机制：一个大任务会被拆分（fork）成多个小任务，待各小任务执行完毕后，再将它们的结果合并（join）成最终的结果。因此，“ForkJoinPool”这一名称直观地反映了其核心功能——在一个线程池内，通过拆分和合并任务，高效地处理那些可递归分解的问题。

#### `CompletableFuture` 有两个版本的方法

1. 同步版本：上一个任务完成后，立即在完成该任务的线程中执行后续回调。

2. 异步版本：上一个任务完成后，将回调任务提交到线程池，由线程池中的其它线程来执行。

#### `CompletableFuture` 和 JavaScript 的异步有什么区别？

JavaScript 的异步是指主线程遇到异步任务时，会将它们丢给浏览器（或 Node.js）的后台线程（Web API）去执行，这些后台线程是可以并发并行执行的。主线程继续执行业务代码，后台线程执行完之后把回调函数（带着结果）塞回任务队列里，当主线程空闲时，会从任务队列中一个个取出回调函数执行。

`CompletableFuture` 会把任务提交到线程池，让线程池里的线程去执行。等任务执行完，再把结果填回 `CompletableFuture` 容器，供后续使用。

#### 回调函数的概念

1. 回调函数，把一个函数作为参数传递给另一个函数，这个被传递的函数将在某个事件（比如异步任务完成）发生后被自动调用。

2. 回调机制允许任务在完成后主动调用回调函数，通知调用者任务已经结束，而不需要调用者去轮询（不断询问）任务的状态。

3. 回调函数通常会带上任务的结果或错误信息，便于调用者进行后续处理。

### 异步创建任务

#### supplyAsync()

静态方法，有返回值，提交一个无参有返回值的方法（`Supplier<U>`，`T get();`）异步执行，执行完任务之后把结果塞进`CompletableFuture`对象里交给下一个步骤。

执行一些需要返回结果的异步任务（如调用远程服务，查询数据库等）。

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // 耗时操作...
    return "返回的结果";
});
```

#### runAsync()

静态方法，没有返回值，提交一个无参无返回值的方法（`Runnable`，`void run();`）异步执行。

适用场景：执行只需要跑一段异步逻辑、不关心返回结果的任务。

```java
CompletableFuture<Void> futureVoid = CompletableFuture.runAsync(() -> {
    // 耗时操作...
});
```

### 链式处理（基于前一个任务的结果）

#### thenApply()

有返回值，接受一个有参有返回值的方法（`Function<T, R>`，`R apply(T t);`）。

获取上一步的结果进行进一步计算，有异常不执行下一步。

```java
CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {  
    return "haha";  
}).thenApply(s -> {  
    System.out.println(s);  
    return "xixi";  
});  
System.out.println(completableFuture.join());
```

#### thenAccept()

无返回值，接受一个有参无返回值的方法（`Consumer<T>`，`void accept(T t);`）只消费上一步的结果，不再往下传递值。

```java
CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {  
    return "haha";  
}).thenAccept(System.out::println);
```

#### thenRun()

无返回值，接收一个无参无返回值的方法（`Runnable`），不获取上一步的结果。

前一步执行完后，再追加一个无关的后续任务。

```java
CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {  
    return "haha";  
}).thenRun(() -> System.out.println("xixi"));
```

### 错误处理

#### exceptionally()

在异常时进行兜底，处理异常并返回一个替代值，避免整个流程崩溃。接收一个有参有返回值的方法（`Function<T, R>`）。

没异常的时候，这个步骤就相当于跳过去了；有异常的时候，从发生异常所在步骤的那一行开始，后面的步骤都跳过，直到遇到此方法。

```java
CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {  
    return "haha";  
}).thenApply(s -> {  
    int a = 1 / 0;  
    return "mimi";  
}).thenApply(s -> {  
    return "wowo";  
}).exceptionally(e -> "xixi").thenAccept(System.out::println);
```

#### handle()

有返回值，接受双参数有返回值方法（`BiFunction<result, Throwable, U>`），无论正常完成还是异常完成都会进入此方法，需要根据是否异常分别执行不同逻辑。它处理完异常，后面的步骤就收不到异常了。

```java
public void basicUsage() throws Exception {  
    CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {  
        return "haha";  
    }).thenApply(s -> {  
        int a = 1 / 0;  
        return "mimi";  
    }).handle((r, e) -> {  
        if (e != null) {  
            return "uu";  
        } else {  
            return "ww";  
        }  
    }).exceptionally(e -> "xixi").thenAccept(System.out::println);  
}
```

### 结果获取

#### get()

阻塞等待结果。声明了会抛出检查异常，调用这个方法要显式处理异常。

#### join()

阻塞等待结果。不声明检查异常，不用显式处理异常。

### 组合多个异步任务

#### allOf()

静态方法，用来将多个独立的异步任务组合在一起，当所有这些任务都执行完毕时，它返回的 `CompletableFuture` 也会完成。这对于等待一组并发任务全部完成后再继续后续处理非常有用。

返回的 `CompletableFuture` 的结果类型是 `Void`，它本身不会收集各个任务的返回值，所以如果需要获取各个任务的结果，通常需要在所有任务完成后分别调用各个任务的 `join()` 或 `get()` 方法。

```java
// 创建多个异步任务  
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {  
    try {  
        Thread.sleep(500);  
    } catch (InterruptedException e) {  
    }  
    return "结果1";  
});  
  
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> {  
    try {  
        Thread.sleep(700);  
    } catch (InterruptedException e) {  
    }  
    return 123;  
});  
  
// 等待所有任务完成  
CompletableFuture<Void> allFutures = CompletableFuture.allOf(future1, future2);  
  
// 阻塞等待所有任务完成  
System.out.println(allFutures.join());  
  
// 获取各个任务的结果  
String result1 = future1.join();  
Integer result2 = future2.join();  
  
System.out.println("future1 的结果: " + result1);  
System.out.println("future2 的结果: " + result2);
```

#### anyOf()

只要传入的多个 `CompletableFuture` 中任意一个完成，就立即返回一个新的 `CompletableFuture`。

返回的 `CompletableFuture` 的结果类型是 `Object`，因此如果任务返回值类型不同，需要根据实际情况进行类型转换。

当你希望采用多个任务中的第一个完成结果（例如多个数据源中，哪个先返回就采用哪个结果）。

适用于多个任务竞速，只要任意一个任务完成就立即返回。

```java
// 创建两个异步任务，其中一个完成较快  
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {  
    try {  
        Thread.sleep(300);  
    } catch (InterruptedException e) {  
    }  
    return "快速返回结果";  
});  
  
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {  
    try {  
        Thread.sleep(600);  
    } catch (InterruptedException e) {  
    }  
    return "较慢返回结果";  
});  
  
// 任一任务完成即可返回  
CompletableFuture<Object> anyFuture = CompletableFuture.anyOf(future1, future2);  
  
// 获取第一个完成的任务结果（需要注意返回类型是 Object，如确定类型可以强转）  
Object result = anyFuture.join();  
System.out.println("第一个完成的任务返回: " + result);
```

#### thenCompose()

用于处理依赖关系较强的异步任务。即第一个任务完成后，其结果作为参数传递给另一个异步任务，避免出现嵌套的 `CompletableFuture<CompletableFuture<T>>`。

当 `thenApply` 返回的是一个普通值，结果会直接封装成一个新的 `CompletableFuture`，如果函数返回的是一个 `CompletableFuture`，则会产生嵌套的 `CompletableFuture<CompletableFuture<T>>`。

当第二个任务需要依赖第一个任务的结果进行异步处理时使用。例如：先查询用户信息，再根据用户信息调用其他异步服务。

使用 `thenApply` 的情况：

```java
CompletableFuture<CompletableFuture<String>> nestedFuture = CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(result -> CompletableFuture.supplyAsync(() -> result + ", World!"));

// 需要额外调用 join() 来获取内部的 CompletableFuture 的结果
String result = nestedFuture.join().join();
System.out.println(result);  // 输出 "Hello, World!"
```

使用 `thenCompose` 的情况：

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> "Hello")
    .thenCompose(result -> CompletableFuture.supplyAsync(() -> result + ", World!"));

System.out.println(future.join());  // 输出 "Hello, World!"
```

当后续操作也是异步操作时，使用 `thenCompose` 更加简洁和直观。

#### thenCombine()

用于并行执行两个独立的异步任务，等它们都完成后，将各自的结果合并成一个最终结果。

当你需要同时调用两个独立服务或执行两个异步任务，然后把它们的结果一起处理（如合并数据、汇总结果等）。

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {  
    return "haha";  
});  
  
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {  
    return "xixi";  
});  
  
future1.thenCombine(future2, (x, y) -> x + " " + y).thenAccept(System.out::println);
```
