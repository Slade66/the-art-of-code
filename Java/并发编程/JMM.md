---
创建日期: 2025-04-13
---

### 是什么？

JMM（Java Memory Model，全称 Java 内存模型）是 Java 虚拟机规范中的并发部分，本质上是一套由官方制定的规则，用于定义 JVM 在并发环境下的行为。它的核心目的是屏蔽底层硬件和操作系统的差异，确保 Java 多线程程序在任何平台、任何 JVM 上都能表现出一致、正确、可预期的执行结果。JMM 就像一份产品经理写给开发者的功能需求文档，明确规定了 JVM 在处理并发时必须遵循的一组语义规则。只有 JVM 实现者严格遵守这些规则，Java 程序员才能在其基础上编写出安全可靠的并发程序。

### 内存模型

在 Java 内存模型中，内存结构被划分为主内存和工作内存：
- 主内存（Main Memory）是所有线程共享的区域，所有共享变量都存储在这里。
- 每个线程都有自己的工作内存（Working Memory），用于存放从主内存中拷贝来的变量副本。

线程对变量的所有读写操作，必须在自己的工作内存中进行，不能直接操作主内存。当一个线程需要使用某个共享变量时，会先从主内存拷贝到自己的工作内存中进行处理，处理完成后再将结果刷新回主内存。

### 可见性

可见性是指：一个线程对共享变量的修改，何时能被其他线程看见并读取到最新值。

在 Java 内存模型（JMM）中，线程的读写操作并不是直接在主内存中进行，而是通过线程自己的工作内存来完成。这种设计带来了可见性问题：一个线程对共享变量的修改，另一个线程可能看不到。

线程工作的流程：
1. 读取共享变量：从主内存拷贝数据到线程的工作内存
2. 执行操作：在线程的工作内存中对副本进行读写
3. 写回共享变量：将修改结果从工作内存刷新回主内存

由于变量在主内存和工作内存之间存在副本关系，这就会导致一个线程修改了变量的值，但由于没有及时同步到主内存，其它线程仍然读取到旧值。

Java 提供了以下几种关键字，来保障变量的可见性：
- `volatile`：把一个变量声明为 `volatile`，JMM 会保证：写入时立刻刷新到主内存，读取时总是从主内存获取最新值，这样其他线程就能看到最新的变化。同时，`volatile` 还会禁止对这个变量的读写指令重排序，保证一定的执行顺序。但它不保证原子性，像 `count++` 这种操作仍然会出现线程安全问题。
- `synchronized` 不仅能实现线程之间的互斥访问，还能保证变量的可见性。加锁时，线程必须先从主内存读取最新的变量值；解锁时，线程对变量的修改也必须刷新回主内存，从而确保其线程它能看到最新的数据。

###  有序性

有序性指的是程序的执行顺序是否与代码的编写顺序保持一致。虽然我们通常从上到下编写代码，但为了优化性能，JVM、编译器以及 CPU 可能会对指令进行重排序。这种重排序在单线程环境下通常不会出错，但在多线程中可能引发意想不到的问题。例如，一个线程尚未完成变量的初始化，另一个线程却提前访问了该变量，可能会读取到一个尚未完全构造的“半初始化”状态。

为了兼顾性能与正确性，Java 内存模型（JMM）允许一定程度的指令重排序，但同时通过 happens-before 原则对关键操作之间的顺序加以约束，从而防止乱序执行带来的 Bug。

happens-before 规则：
1. 程序顺序规则：同一个线程里，写在前面的代码先执行。
2. 锁规则（`synchronized`）：一段代码在释放锁之后，其它代码在获取同一把锁时，能够看到释放锁前所做的所有修改。
3. `volatile` 规则：一个线程改了 `volatile` 变量，其它线程立马就能看到这个新值。
4. 线程启动规则：调用了 `start()` 后，子线程才会开始跑，里面的代码不会早于 `start()` 执行。
5. 线程终止规则：当你 `join()` 等待的线程执行完毕后，它在运行期间所做的修改，你在 `join()` 之后的代码里都能看到。
6. 传递性规则：如果 A 先于 B，B 又先于 C，那 A 也一定先于 C，顺序能传下去。
