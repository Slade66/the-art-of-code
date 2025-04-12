---
创建日期: 2025-04-12
---

`PriorityBlockingQueue` 是 Java 并发包中的一种线程安全、基于优先级的无界阻塞队列，其元素的出队顺序由优先级决定，而非插入顺序（非 FIFO），非常适用于需要按任务优先级处理的多线程场景。它的底层采用数组形式实现的小顶堆结构，元素默认按照 `Comparable` 接口或自定义的 `Comparator` 进行从小到大的优先级排序，并且支持自动扩容。

### 常用方法

#### 构造方法

```java
PriorityBlockingQueue() // 默认初始容量为 11，元素必须实现 Comparable 接口
PriorityBlockingQueue(int initialCapacity) // 指定初始容量，但仍是无界队列（自动扩容）
PriorityBlockingQueue(int initialCapacity, Comparator<? super E> comparator) // 使用自定义 Comparator 排序
```

#### 添加元素

```java
boolean offer(E e) // 非阻塞添加，成功返回 true，始终不会因“满”而失败
void put(E e) throws InterruptedException // 理论阻塞添加，但因为是无界队列，实际不会阻塞
boolean add(E e) // 添加元素，成功返回 true，失败抛异常（和 offer 行为一致）
```

- 所有添加方法都不会阻塞，因为队列是无界的。
- 元素必须实现 `Comparable` 或提供 `Comparator`，否则添加时报 `ClassCastException`。

#### 获取元素

```java
E take() throws InterruptedException // 阻塞获取，如果队列为空，调用线程会等待直到有元素
E poll() // 非阻塞获取，没有元素就返回 null
E poll(long timeout, TimeUnit unit) throws InterruptedException // 限时阻塞获取，超时还没数据就返回 null
E peek() // 查看队首元素，不移除，队列为空时返回 null
```
