---
创建日期: 2025-04-12
---

`ArrayBlockingQueue` 是 Java 并发包 `java.util.concurrent` 中的一个有界阻塞队列，它实现了 `BlockingQueue` 接口，底层由数组实现，遵循先进先出（FIFO）原则，线程安全，它能自动让线程在数据满/空时等待或唤醒，适合用在多线程间传递任务或数据。

### 常用方法

#### 构造方法

```java
ArrayBlockingQueue(int capacity)
ArrayBlockingQueue(int capacity, boolean fair)
```

#### 添加元素

```java
void put(E e) throws InterruptedException // 阻塞添加，当队列已满时，调用线程会阻塞直到有空位
boolean offer(E e) // 非阻塞添加，添加成功返回 true，满了立即返回 false
boolean offer(E e, long timeout, TimeUnit unit) // 限时阻塞添加，超时还没空位就返回 false
boolean add(E e) // 立即插入，失败抛异常（IllegalStateException）
```

#### 获取元素的方法

```java
E take() throws InterruptedException // 阻塞获取，如果队列为空，阻塞直到有数据
E poll() // 非阻塞获取，没有就返回 null
E poll(long timeout, TimeUnit unit) throws InterruptedException // 限时阻塞获取，超时还没数据就返回 null
E peek() // 查看队首元素，不会移除，空时返回 null
```
