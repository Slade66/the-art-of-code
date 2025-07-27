- `map` 是 Go 中用于存储键值对的数据结构，是一种无序的集合。
  title:: Go 的 map
- 与数组或切片相比，`map` 在查找元素时通常具有常数时间复杂度 `O(1)`，而数组或切片的查找复杂度为 `O(n)`。
- 只要键类型可以比较（如基本类型），就可以作为 `map` 的键。
- 声明 `map` 类型的变量
  heading:: true
	- ```go
	  var 变量名 map[键类型]值类型
	  ```
	- 此时只声明了变量，并未初始化，映射变量的默认值为 `nil`，不能直接添加键值对，否则会导致运行时错误。
	- 从 `nil` map 中读取数据是安全的：读取键值时会返回值类型的零值，获取长度返回 0，使用 `for-range` 遍历时不会执行循环。这些行为与空的 `map` 一致，不会引发 panic。
- 初始化 `map`
  heading:: true
	- 使用 `make` 函数或字面量来创建实际的 `map` 实例。
	- 先声明再赋值：
		- ```go
		  var m map[string]int
		  m = make(map[string]int)
		  ```
	- 一步到位：
		- ```go
		  var m = make(map[string]int)
		  ```
	- 使用短变量声明：
		- ```go
		  m := make(map[string]int)
		  ```
- 使用映射字面量创建 `map`
  heading:: true
	- 如果预先知道映射的键和值，你可以使用字面量来创建映射。
	- 这种方式无需使用 `make` 函数。
	- ```go
	  m := map[string]int{"a": 1, "b": 2, "c": 3}
	  m := map[string]int{} // 空映射
	  ```
- 访问 `map`
  heading:: true
	- 访问 `map` 中的键时，可以选择获取一个额外的布尔值，用于判断该键是否存在。
	- ```go
	  value, ok := m["key"]
	  ```
	- 如果键存在，`ok` 为 `true`，否则为 `false`。
	- 如果访问不存在的键，不会报错，而是返回该值类型的零值。
- 删除键值对
  heading:: true
	- 使用 `delete` 函数删除指定的键：
	- ```go
	  delete(目标映射, "要删除的键")
	  ```
- 遍历 `map`
  heading:: true
	- ```go
	  for key, value := range m {
	      fmt.Println(key, "=>", value)
	  }
	  ```
	- 遍历顺序是随机的，每次执行的结果可能不同。
- `map` 是引用类型
  heading:: true
	- 当你将一个 `map` 变量赋值给另一个变量，或者将 `map` 作为参数传递给函数时，你传递的只是一个指向底层数据结构的“指针”。
	- 这意味着在函数内部对 `map` 的修改会影响到函数外部的原始 `map`。
- `map` 不是并发安全的
  heading:: true
	- 如果你有多个 goroutine 同时对一个 map 进行读写操作，程序会直接崩溃。
	- 解决方法：
		- 使用 `sync.RWMutex`（读写互斥锁）。
		- 使用专门为并发场景设计的 `sync.Map` 类型。
-