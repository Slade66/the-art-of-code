- **基本概念：**
  collapsed:: true
	- `map` 是 Go 中用于存储键值对的内置数据结构，是一种无序的集合。
	- 与数组或切片相比，`map` 在查找元素时通常具有常数时间复杂度 `O(1)`，而数组或切片的查找复杂度为 `O(n)`。
	- 只要键类型可以比较（如基本类型），就可以作为 `map` 的键。
	- `map` 建立了一种**映射关系**。它通过一个唯一的 `Key`，能快速找到对应的 `Value`。
- **创建映射：**
  collapsed:: true
	- **语法：**`map[KeyType]ValueType`
	- **声明 `map` 类型的变量但不初始化：**
		- **语法：**
			- ```go
			  var 变量名 map[键类型]值类型
			  ```
			- 此时只声明了变量，并未初始化，映射变量的默认值为 `nil`，即它并不指向任何有效的底层数据结构。
	- **初始化 `map`：**
		- 为了能够向 `map` 添加键值对，必须在使用 `map` **之前**，使用 `make()` 函数初始化它。
		- ```go
		  var m map[string]int
		  m = make(map[string]int)
		  ```
		- 执行完这行代码后，`m` **不再是 `nil`**。它现在是一个**空**的、但**已经准备好接收数据**的 `map`。
	- **一步到位：**
		- ```go
		  var m = make(map[string]int)
		  ```
	- **使用短变量声明：**
		- ```go
		  m := make(map[string]int)
		  ```
	- **使用字面量创建映射：**
		- 如果预先知道映射的键和值，在创建 `map` 的同时，就用键值对把它填满。
		- `map` 字面量的语法就是 `map类型` 后面跟一对花括号 `{}`, 里面包含任意数量的 `Key: Value` 键值对，用逗号 `,` 隔开。
		- ```go
		  m := map[string]int{"a": 1, "b": 2, "c": 3}
		  m := map[string]int{} // 空映射
		  ```
- **访问映射中的键值对：**
  collapsed:: true
	- 通过 `map[key]` 可以根据键获取对应的值。如果访问一个不存在的键，不会报错，而是返回该值类型的零值。
	- **检查键是否存在**
		- 访问 `map` 中的键时，可以选择获取一个额外的布尔值，用于判断该键是否存在。
		- ```go
		  value, ok := m["key"]
		  ```
		- 如果键存在，`ok` 为 `true`，`elem` 是对应的值；否则为 `false`，`elem` 是零值。
- **删除键值对：**
  collapsed:: true
	- `delete()`：
		- 可以使用内置的 `delete()` 函数来删除映射中的某个键值对。
		- 如果 `key` 不存在，这个操作**不会做任何事，也不会报错**。
		- ```go
		  delete(目标映射, "要删除的键")
		  ```
	- `clear()`：
		- 删除 map 里的全部键值对。
- **遍历映射：**
  collapsed:: true
	- `range` 可以用于遍历映射，它返回键和值的每一对。
	- ```go
	  for key, value := range m {
	      fmt.Println(key, "=>", value)
	  }
	  ```
	- 遍历顺序是随机的，每次执行的结果可能不同。
- **插入或更新键值对：**
  collapsed:: true
	- 通过 `map[key] = value` 的语法来设置映射中的键值对。
	- 如果 `key` 不存在，就创建它；如果 `key` 已存在，就覆盖它的值。
	- **示例：**
		- ```go
		  m["k1"] = 7
		  m["k2"] = 13
		  ```
- **`map` 是引用类型：**
  collapsed:: true
	- **底层结构**：Go 的 map 在底层是一个指向 `hmap`（Hash Map）结构体的指针。
	- 当你创建一个 map 时，你拿到的变量本质上是一个指针。
	- 当你把 map 传递给一个函数时，你拷贝的是这个指针。
	- 函数内部对 map 的修改（增加、删除、修改键值对），会影响到外部原本的 map。
- **`map` 不是并发安全的：**
  collapsed:: true
	- 如果你有多个 goroutine 同时对一个 map 进行读写操作，程序会直接崩溃。
	- **办法：**
		- 使用 `sync.RWMutex`（读写互斥锁）。
		- 使用专门为并发场景设计的 `sync.Map` 类型。
- **操作未初始化的 Map**
  collapsed:: true
	- **只声明，不 make**
		- ```go
		  var m map[string]int // 这是一个 nil map
		  
		  fmt.Println(m["key"]) // ✅ 读是可以的，返回零值 0
		  delete(m, "key")      // ✅ 删也是可以的（什么都不发生）
		  
		  m["key"] = 1          // ❌ Panic! (assignment to entry in nil map)
		  ```
		- `var m` 只是声明了一个指针，它指向 `nil`。你没有创建底层的 `hmap` 结构，所以没有地方存数据。
		- map 的零值是 `nil`。
		- **从 `nil` map 中读取数据是安全的：**读取键值时会返回值类型的零值，获取长度时返回 0，使用 `for-range` 遍历时不会执行循环。这些行为与空 `map` 相同，不会引发 panic。
		- 不能向一个 `nil` map 添加键值对，否则会导致运行时错误。
	- 使用 map 前必须使用 `make` 初始化，或者使用字面量 `{}`。
		- ```go
		  m := make(map[string]int)
		  // 或者
		  m := map[string]int{}
		  ```
- len 函数返回键值对的个数。
- maps.Equal 函数判断两个 map 的元素是否一样。
-