---
创建日期: 2025-04-03
---

#### 方式一：继承 `Thread` 类

1. 继承 `Thread` 类
2. 重写 `run` 方法
3. 创建子类对象
4. 调用 `start()`

```java
class MyThread extends Thread {
	@Override
	public void run() {
		for (int i = 1; i <= 10; i++) {
			System.out.println(i);
		}
	}
}

@Test
public void testCreateThread1() {
	new MyThread().start();
}
```

Java 只支持单继承，也就是说一个类只能继承一个父类。如果继承了 `Thread` 类，就无法再继承其他类，因此这种写法具有一定的局限性。

为什么要重写 `run` 方法？因为线程要执行的任务就定义在这个方法中。

为什么要调用 `start` 方法？因为直接调用 `run` 方法只是普通的方法调用，并不会启动新线程。只有调用 `start` 方法，JVM 才会创建新线程，并在该线程中执行 `run` 方法中的代码。

#### 方式二：实现 `Runnable` 接口

1. 用一个类实现 `Runnable` 接口
2. 重写 `run` 方法
3. 创建 `Thread` 类的对象
4. 把第一步的实现类对象传入第三步的对象里
5. 调用 `start()`

```java
class MyThread2 implements Runnable {
	@Override
	public void run() {
		for (int i = 1; i <= 10; i++) {
			System.out.println(i);
		}
	}
}

@Test
public void testCreateThread2() {
	new Thread(new MyThread2()).start();
}
```

Java 允许一个类实现多个接口，从而有效弥补了单继承的局限性。

这种方式与方式一创建的线程在本质上是一样的，都是通过调用本地方法 `start0` 来启动新线程。

#### 方式三：实现 `Callable` 接口

使用这种方式创建的线程可以返回结果，该结果可以通过 `FutureTask` 获取。

1. 用一个类实现 `Callable` 接口，泛型中传入线程的返回值类型
2. 重写 `call` 方法
3. 创建一个 `FutureTask` 对象，并将第一步中实现的任务类作为参数传入，这是为了能在未来获取线程的执行结果
4. 创建 `Thread` 类对象，把第三步中创建的 `FutureTask` 对象作为参数传入
5. 调用 `start()`

```java
class MyThread3 implements Callable<Integer> {
	@Override
	public Integer call() throws Exception {
		int sum = 0;
		for (int i = 1; i <= 10; i++) {
			sum += i;
		}
		return sum;
	}
}

@Test
public void testCreateThread3() {
	FutureTask<Integer> futureTask = new FutureTask<>(new MyThread3());
	new Thread(futureTask).start();
	try {
		Integer result = futureTask.get();
		System.out.println(result);
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```

#### 方式四：使用 `ExecutorService`（线程池）

```java
class MyThread4 implements Runnable {
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println(i);
		}
	}
}

@Test
public void testCreateThread4() {
	ExecutorService pool = Executors.newFixedThreadPool(3);
	pool.submit(new MyThread4());
	pool.shutdown();
}
```

这种方式可以复用已有线程，避免频繁创建和销毁线程所带来的开销；同时还能控制线程数量，有效防止系统资源被耗尽。
