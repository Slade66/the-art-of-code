---
创建日期: 2025-04-12
---

`SynchronousQueue` 是 Java 并发包（java.util.concurrent）中一个特殊的阻塞队列。它没有容量，不能缓存任何元素，任何一次 `put()` 操作都必须等待另一个线程执行 `take()`，反之亦然。它的核心作用是在线程之间实现一对一的直接数据交换。

### 常用方法

#### 放入元素

```java
void put(E e) throws InterruptedException; // 阻塞当前线程直到有消费者 take()
boolean offer(E e); // 非阻塞：尝试交付，如果当前没有消费者线程调用 take()，立即失败。true 表示成功交付，false 表示失败
```

#### 取出元素

```java
E take() throws InterruptedException; // 阻塞当前线程直到有生产者 put()
E poll(); // 非阻塞：尝试获取数据，如果当前没有生产者线程 put()，立即失败。返回值：成功返回元素，失败返回 null
```

