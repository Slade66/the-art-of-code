---
创建日期: 2025-04-11
---

### 有什么用？

**HashMap 的线程安全问题**

`HashMap` 本身并非线程安全。在多线程环境下，当多个线程并发地对同一个 `HashMap` 实例执行修改操作（例如 `put` 或 `remove`）时，可能会引发数据覆盖问题：由于多个线程同时尝试写入相同的内存位置，后执行写入的线程可能会覆盖先前线程的写入结果。

**传统同步方式的性能瓶颈**

虽然可以通过 `Collections.synchronizedMap(new HashMap())` 创建线程安全的 Map，但这种方式存在明显的性能瓶颈。它采用装饰器模式（Decorator Pattern）包裹原生的 Map，在每个方法的外层手动加上 `synchronized` 实现线程安全。这样一来，在任意时刻只能有一个线程访问该 Map，其它线程必须等待锁的释放。在高并发场景下，这种全局同步机制会导致显著的性能下降。

```java
public static <K, V> Map<K, V> synchronizedMap(Map<K, V> m) {
	return new SynchronizedMap<>(m);
}

private static class SynchronizedMap<K, V> implements Map<K, V>, Serializable {
	private final Map<K, V> m; // 被包装的 Map
	final Object mutex;      // 同步锁对象（默认是 this）

	SynchronizedMap(Map<K, V> m) {
		this.m = Objects.requireNonNull(m);
		this.mutex = this;
	}

	public V put(K key, V value) {
		synchronized (mutex) {
			return m.put(key, value);
		}
	}

	public V get(Object key) {
		synchronized (mutex) {
			return m.get(key);
		}
	}

	// ... 所有方法都加锁
}
```

`Hashtable` 是 JDK 早期提供的一种线程安全的哈希表实现，它通过在所有方法上使用 `synchronized` 关键字来实现同步。虽然这种方式能保证线程安全，但由于每个操作都对整个对象加锁，锁的粒度大，从而导致并发性能极差，难以满足高并发场景的需求。

**ConcurrentHashMap 的线程安全与并发优势**

`ConcurrentHashMap` 是 Java 并发包中提供的一种线程安全的哈希表实现。它通过一系列精巧的机制（如 JDK 7 中的分段锁，JDK 8 及以后版本中的更细粒度锁和 CAS 操作），在确保数据正确性的同时实现了高并发性能。与传统的同步方式相比，`ConcurrentHashMap` 允许多个线程同时读取数据，并在修改时仅对部分数据加锁，而非锁定整个表，从而显著提升了并发读写效率。如果你需要在多线程程序中使用哈希表来存储数据，并且希望保证数据的安全性和程序的性能，那么 `ConcurrentHashMap` 就是你的首选。

### 怎么用？

**创建 ConcurrentHashMap 实例**

- `new ConcurrentHashMap<>()`：创建一个具有默认初始容量（16）和默认加载因子（0.75）的 `ConcurrentHashMap`。
- `new ConcurrentHashMap<>(int initialCapacity)`：创建一个具有指定初始容量的 `ConcurrentHashMap`。
- `new ConcurrentHashMap<>(int initialCapacity, float loadFactor)`：创建一个具有指定初始容量和加载因子的 `ConcurrentHashMap`。加载因子决定了何时进行扩容。

**常用的方法**

- `put(K key, V value)`：将指定的键值对插入 Map 中，如果 key 已存在，则覆盖原值。
- `get(Object key)`：返回与指定 key 关联的值，若不存在则返回 `null`。
- `remove(Object key)`：移除指定 key 的键值对，若存在则删除并返回对应值。
- `containsKey(Object key)`：判断指定 key 是否存在于 Map 中。
- `putIfAbsent( K key, V value)`：仅当 key 不存在时才插入值。如果已存在对应的 key，则保持原值不变。
- `size()`：返回 Map 中键值对的数量。
- `isEmpty()`：判断 Map 是否为空。
