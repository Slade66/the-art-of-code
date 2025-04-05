---
创建日期: 2025-04-05
---

#### 作用一：保证变量的可见性

每个线程都有自己的工作内存（也可以理解为本地缓存），用于存储从主内存（即进程的共享内存）中拷贝来的变量副本。线程对变量的读取和写入，通常都是在自己的工作内存中进行的，而不是直接操作主内存。

因此，一个线程对变量所做的修改，不会立即反映到主内存中。同时，其他线程也无法立即看到这个修改，因为它们读取的仍是自己工作内存中缓存的旧值。这就可能导致可见性问题：一个线程更新了变量的值，但其他线程却永远感知不到这个变化，从而引发程序行为异常。

`volatile` 在英语中意为“易变的、不稳定的”。在 Java 中，`volatile` 用于修饰变量，表示该变量的值可能会被多个线程同时访问和修改，因此禁止线程将该变量缓存在工作内存中，强制每次访问时都必须从主内存中读取最新的值。此外，`volatile` 还确保了对该变量的写操作会立刻刷新到主内存中，以便其它线程能够及时看到更新后的值，从而保证变量的变化对所有线程都是立即可见的，避免出现缓存导致的数据不一致问题。

```java
public class VolatileVisibilityDemo {

    private static volatile boolean isRunning = true;

    public static void main(String[] args) {

        Thread worker = new Thread(() -> {
            System.out.println("子线程启动...");
            while (isRunning) {
                // 模拟一些处理逻辑
                // System.out.println(1);
            }
            System.out.println("子线程检测到 isRunning 为 false，退出循环。");
        });

        worker.start();

        try {
            // 主线程等待 2 秒，然后修改 isRunning
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("主线程将 isRunning 设置为 false");
        isRunning = false;
    }

}
```

注意：
1. 当子线程执行 `System.out.println()` 时，JVM 会刷新线程的工作内存，从主内存中重新加载共享变量的值，以避免数据不一致的问题。
2. 当子线程调用 `Thread.sleep()` 进入休眠时，线程会被挂起。在挂起与恢复的过程中，操作系统可能会保存并恢复线程的上下文状态，这一过程同样可能触发线程工作内存与主内存之间的数据同步。

本质上仍然存在数据可见性问题，只是由于调用了某些特殊方法而间接“规避”了问题。正确做法还是：用 `volatile` 或加锁才能在语言层面明确保证变量的可见性，摆脱“看 JVM 心情”的不稳定因素。

#### 作用二：禁止指令重排序

限制指令重排（提升执行可预期性）

底层会插入内存屏障。

双重检验锁实现的线程安全的单例模式：

```java
public class Singleton {

    public static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

在这里使用 `volatile` 关键字修饰 `instance` 变量是非常有必要的，因为对象的创建操作并不是原子性的，它实际上分为以下三个步骤：
1. 为对象分配内存空间
2. 初始化对象
3. 将对象的引用赋值给 `instance` 变量

如果不使用 `volatile`，系统在执行时可能会对这三个步骤进行指令重排序（因为在它看来结果都一样），尤其是将步骤 2 和步骤 3 调换顺序。在单线程环境下这不会有问题，但在多线程环境中就可能出问题。

当多个线程同时进入获取单例的方法时，只有一个线程会获得锁并开始创建对象。如果这个线程在执行到被调换到步骤 2 的步骤 3（即将对象引用赋值给 `instance`）之后、但还未完成被调换到步骤 3 的步骤 2（对象的初始化）时被挂起，此时 `instance` 已经不为 `null` 了。其它线程获取到 CPU 时间片开始继续执行，在判断 `instance != null` 时就会认为对象已经创建完成，直接返回这个尚未初始化完毕的对象。这样返回的就是一个“半初始化”的实例，将会导致非常难以排查的隐蔽性 `bug`。

#### 注意事项

1. `volatile` 必须修饰实例变量或者静态变量，不能修饰本地变量。
2. `volatile` 能保证变量的可见性和有序性，但不保证原子性——还得加锁或使用原子类。
