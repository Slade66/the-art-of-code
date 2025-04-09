---
创建日期: 2025-04-08
---

### 有什么用？

它可以让每个线程都有自己的变量，各自拥有自己独立的一份数据副本.

### 为什么多个线程操作同一个 ThreadLocal 却得到不同的结果？

表面上看，多个线程都在调用同一个 `ThreadLocal` 对象的 `set()` / `get()` 方法，但它们最终读写的数据却互不影响。

这是因为：`ThreadLocal` 只是一个“标识”，它并不真正存储数据，而是作为一个 key，用来在每个线程自己的 `ThreadLocalMap` 中查找或存放对应的值。

每个线程都有一个专属的 `ThreadLocalMap`，当我们执行 `threadLocal.set(value)` 或 `threadLocal.get()` 时，实际上执行了：`Thread t = Thread.currentThread();` ，然后在这个线程 `t` 的 `threadLocals` 中，以当前这个 `ThreadLocal` 实例为 key 进行读写操作。

因为每个线程通过 `Thread.currentThread()` 拿到的是不同的线程对象，它们内部的 `ThreadLocalMap` 也是彼此独立的，所以即使多个线程使用的是同一个 `ThreadLocal` 实例，也不会相互影响，各自只操作自己的那一份数据副本。

### 为什么要用 ThreadLocal 对象作为 key？

`ThreadLocalMap` 中存储的是键值对 (`ThreadLocal` 实例 → 值)，这样设计是为了支持多个 `ThreadLocal` 变量共存。在实际开发中，我们可能会定义多个不同用途的 `ThreadLocal` 变量（比如 `threadLocalA`、`threadLocalB`），分别用于存储不同业务场景下的线程私有数据。由于这些变量都存在于同一个线程的 `ThreadLocalMap` 中，就必须以 `ThreadLocal` 实例本身作为 key，才能正确区分和管理不同的数据条目。

此外，一个 `ThreadLocal` 实例只能对应存储一个值——如果你希望在同一个线程中存储多个线程私有变量，就需要创建多个不同的 `ThreadLocal` 实例。

### 为什么 Entry 的 key 是弱引用？

在 `ThreadLocalMap` 中，key（`ThreadLocal` 对象）是弱引用。这样当外部不再有强引用指向这个 `ThreadLocal` 时，GC 就可以回收它，避免 `ThreadLocal` 对象本身无法被释放，从而造成内存泄漏。

**为什么 key 弱、value 强？**

这是出于实际使用中的“生命周期”考量：key（`ThreadLocal` 对象）通常是方法内创建的临时变量，生命周期较短；value 是业务真正要存储的数据，可能还没用完，不能随便被 GC 回收。所以设计成“key 弱引用、value 强引用”，才能让 `ThreadLocal` 在没有引用时被 GC 回收，而不是反过来导致重要业务数据被错误清理。

### 为什么用完要手动删除值？

在 `ThreadLocalMap` 中，`ThreadLocal` 是作为 弱引用（key） 存在的，而它对应的 value 是强引用。

当某个 `ThreadLocal` 实例没有外部强引用指向时，JVM 会在 GC 时将其回收。此时，`ThreadLocalMap.Entry` 中的 key 就会变为 `null`。

然而，与之关联的 value 仍然是强引用，并不会自动被清理。由于 key 已经变为 `null`，Map 也无法再定位到这条记录，自然也就 无法自动删除这条 entry。

这样就会造成一种“无主对象”的情况：value 虽然还存在，但已经没有任何合法的访问路径，成为了悬挂在内存中的“垃圾”，无法被 GC 回收。

在短生命周期的线程中，这些遗留的数据通常会随着线程的终结一并释放，问题不大；但在线程池或其他长时间存活的线程中，线程不会主动销毁，这些“悬挂”的 value 会持续堆积，导致内存泄漏。更严重的是，后续任务若意外访问这些旧值，可能还会引发业务逻辑错误。

因此，在使用完 `ThreadLocal` 后，一定要主动调用 `remove()` 方法清理掉当前线程中的对应数据，防止资源无法释放或被错误复用。

### 如何跨线程传递 ThreadLocal 的值？

普通的 `ThreadLocal` 无法直接将父线程中的变量传递给子线程，而 `InheritableThreadLocal` 能在创建子线程时，自动拷贝父线程的值，让子线程获得一份“初始值”。

不过，当父线程启动子线程后，如果父线程再次调用 `set()` 设置新值，并不会自动同步到子线程，因为子线程只在初始化阶段进行过一次拷贝。

```java
public class InheritableThreadLocalDemo {

    // 1. 声明一个 InheritableThreadLocal
    private static final InheritableThreadLocal<String> threadLocal = new InheritableThreadLocal<>();

    public static void main(String[] args) {
        // 2. 在父线程中 set 值
        threadLocal.set("Parent Thread's Value");

        // 3. 创建子线程
        new Thread(() -> {
            // 4. 子线程启动后，可以直接 get 到父线程的值（自动拷贝）
            System.out.println("Child Thread Value: " + threadLocal.get());
            
            // 可以修改自己这份本地变量，不影响父线程
            threadLocal.set("Child Thread's Own Value");
            System.out.println("Child Thread Modified Value: " + threadLocal.get());
        }).start();
        
        // 父线程自己的值
        System.out.println("Parent Thread Value: " + threadLocal.get());
    }
}
```
