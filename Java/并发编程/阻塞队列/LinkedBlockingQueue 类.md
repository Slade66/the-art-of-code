---
创建日期: 2025-04-12
---

`LinkedBlockingQueue` 是 Java 并发包中基于链表结构实现的阻塞队列，支持线程安全的元素插入与移除操作，并提供可选的容量限制。当队列已满时，生产者线程会被阻塞；当队列为空时，消费者线程也会等待，因而非常适合用于多线程环境下的生产者-消费者模型。

### 常用方法

#### 构造方法

```java
LinkedBlockingQueue() // 创建一个无界队列，最大容量为 Integer.MAX_VALUE，风险是可能导致内存溢出
LinkedBlockingQueue(int capacity) // 创建一个有界队列，推荐设置容量限制避免 OOM
```

#### 添加元素

```java
void put(E e) throws InterruptedException // 阻塞添加，当队列已满时，调用线程会阻塞直到有空位
boolean offer(E e) // 非阻塞添加，添加成功返回 true，满了立即返回 false
boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException // 限时阻塞添加，超时还没空位就返回 false
boolean add(E e) // 立即插入，失败（队列满）时抛出 IllegalStateException 异常
```

#### 获取元素

```java
E take() throws InterruptedException // 阻塞获取，如果队列为空，调用线程会阻塞直到有数据
E poll() // 非阻塞获取，没有元素就返回 null
E poll(long timeout, TimeUnit unit) throws InterruptedException // 限时阻塞获取，超时还没数据就返回 null
E peek() // 查看队首元素，不会移除，队列为空时返回 null
```
