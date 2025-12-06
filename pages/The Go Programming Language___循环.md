- Go 语言不支持 `while`，只有 `for` 这一种循环关键字、但可以用 `for` 模拟。
- 语法
  heading:: true
	- **注意：**
	  collapsed:: true
		- `for` 语句不需要圆括号，但必须有花括号。
		- 后置语句的多变量更新不能写成 `x++, y++`：
			- 在 Go 语言中：
				- **`++` 和 `--` 是语句，而不是表达式：**
					- 它们只能单独一行，不能与赋值语句或任何其他表达式结合使用。
					- 所以不允许在 `for` 循环的后置语句部分用逗号分隔多条 `x++`、`y++`。
				- **`++` 和 `--` 只能是后缀操作符**：它们只能放在变量后面（例如 `p1++`），但不能放在变量前面（例如 `++p1`，这是不合法的）。
			- **不合法的的写法：**
				- ```go
				  // 下面的写法是：
				  for i, j := 0, 10; i < j; i++, j++ {
				      // …
				  }
				  ```
			- **正确的做法是使用并行赋值语法：**
				- 这里 `i, j = i+1, j-1` 是一个赋值语句，允许同时更新多个变量。
				- ```go
				  for i, j := 0, 10; i < j; i, j = i+1, j-1 {
				      // …
				  }
				  ```
	- **经典三段式：**
	  collapsed:: true
		- ```go
		  for 初始化语句; 条件表达式; 后置语句（更新） {
		      // 循环体
		  }
		  ```
		- **for 语句的执行流程：**
			- 初始化语句在第一次迭代前执行，只会执行一次。
			- 条件表达式在每次循环体（`{...}` 内的代码）执行之前评估：
				- 如果为 `true`，就执行循环体。
				- 如果为 `false`，循环将立即停止，程序会跳到 `for` 循环的右花括号 `}` 之后的代码继续执行。
			- 后置语句在每次循环体（`{...}` 内的代码）执行之后才会被执行。
	- **for 循环的变体 1：模拟 while 循环**
	  collapsed:: true
		- 省略初始化语句和后置语句，只写一个条件，`for` 循环就等价于 `while` 循环。
		- ```go
		  for 条件表达式 {
		      // 循环体
		  }
		  ```
	- **for 循环的变体 2：无限循环**
	  collapsed:: true
		- `for` 后面什么都不写，表示一个死循环，等价于 `while(true)`。
		- **注意：**你必须在循环体内部使用 `break` 关键字来显式地跳出循环。
		- ```go
		  for {
		      // 无限循环体
		  }
		  ```
	- **for 循环的变体 3：遍历集合（使用 `:= range`）：**
	  collapsed:: true
		- 对于切片（slice）、数组（array）、字符串（string）、map、channel，Go 提供了 `for … range` 的语法访问其中的每个元素。
		- | **数据类型** | **第 1 个返回值 (Key)** | **第 2 个返回值 (Value)** | **特点** |
		  | ---- | ---- | ---- |
		  | **数组 / 切片** | **索引** (Index) | **元素的值** (Copy) | 最常用的场景 |
		  | **Map** | **键** (Key) | **值** (Value) | **注意：遍历顺序是随机的！** |
		  | **字符串** | **索引** (Pos) | **字符** (Rune) | 自动按 UTF-8 字符解码，支持中文 |
		  | **Channel** | **元素值** | *(无)* | 一直读直到通道关闭 (close) |
		  | **自定义函数** | *(自定义)* | *(自定义)* | *Go 1.23+ 新特性 (迭代器)* |
		- **遍历数组或切片：**
			- ```go
			  nums := []int{10, 20, 30}
			  
			  // 写法 1：既要索引，也要值
			  for i, v := range nums {
			      fmt.Printf("下标: %d, 值: %d\n", i, v)
			  }
			  
			  // 写法 2：只需要值（用 _ 丢弃索引）
			  for _, v := range nums {
			      fmt.Println("值:", v)
			  }
			  
			  // 写法 3：只需要索引（直接不写第二个变量）
			  for i := range nums {
			      fmt.Println("下标:", i)
			  }
			  ```
			- 每次迭代，`range` 会返回两个值：索引和该索引处的元素。即使只使用其中一个值，另一个也必须写出来。
			- ((683ea645-6f46-4697-ae64-304e5ef19ea3)) 解决办法是使用空标识符 `_` 来丢弃不需要的变量。
		- **遍历 Map：**
			- **注意：遍历顺序是随机的！**
			- ```go
			  m := map[string]string{"name": "Gemini", "role": "AI"}
			  
			  for k, v := range m {
			      fmt.Printf("%s -> %s\n", k, v)
			  }
			  ```
		- **遍历字符串：**
			- `range` 很聪明，它不是按字节遍历，而是按**字符**（rune）遍历。
			- ```go
			  for i, c := range "Go语言" {
			      // i 是字节位置，c 是字符本身
			      fmt.Printf("%d: %c\n", i, c)
			  }
			  // 输出：
			  // 0: G
			  // 1: o
			  // 2: 语  (注意：下一个索引直接跳到了 5，因为“语”占了 3 个字节)
			  // 5: 言
			  ```
		- **遍历整数：**
			- 让你不再需要写 `for i := 0; i < n; i++` 这种啰嗦代码。这是对传统计数循环的语法糖。
			- **before：**
				- ```go
				  for i := 0; i < 10; i++ {
				      fmt.Println(i)
				  }
				  ```
			- **after：**
				- 你直接 `range` 一个整数 `n`，它就会从 `0` 遍历到 `n-1`。
				- 如果 n <= 0，循环体一次都不会执行。
				- ```go
				  // 遍历 0 到 9
				  for i := range 10 {
				      fmt.Println(i)
				  }
				  ```
		- **遍历函数：**
			- 让你可以自定义“迭代器”，用 `for range` 遍历任何东西（链表、树、无限序列等），而不仅仅是切片和 Map。
			- 这个模式最美妙的地方在于封装。假如未来你把底层的单向链表改成了双向链表、数组或者树，只要你修改了 `All()` 方法内部的逻辑，外部调用者的 `for-range` 代码一行都不用改。
			- **遍历单向链表：**
				- 使用 Go 1.23 的 `range` 迭代器，你可以把链表复杂的“指针移动逻辑”封装起来，让外部调用者像遍历切片一样轻松。
				- ```go
				  package main
				  
				  import "fmt"
				  
				  // 1. 定义链表节点
				  type Node struct {
				      Val  int
				      Next *Node
				  }
				  
				  // 2. 定义链表结构
				  type LinkedList struct {
				      Head *Node
				  }
				  
				  // 3. 核心：实现迭代器方法
				  // 命名为 All 是参考标准库 slices.All 的习惯
				  func (ll *LinkedList) All() func(yield func(int) bool) {
				      return func(yield func(int) bool) {
				          // 这里的逻辑只负责“怎么走”
				          for curr := ll.Head; curr != nil; curr = curr.Next {
				              // 将当前节点的值"推"出去
				              if !yield(curr.Val) {
				                  return // 如果外部 break 了，这里也要停止
				              }
				          }
				      }
				  }
				  
				  // 辅助函数：快速添加节点
				  func (ll *LinkedList) Add(val int) {
				      newNode := &Node{Val: val, Next: ll.Head}
				      ll.Head = newNode
				  }
				  
				  func main() {
				      // 初始化链表: 30 -> 20 -> 10 -> nil
				      list := &LinkedList{}
				      list.Add(10)
				      list.Add(20)
				      list.Add(30)
				  
				      fmt.Println("开始遍历链表：")
				  
				      // 4. 使用 range 遍历
				      // 注意：外部完全不需要知道 Next 指针的存在！
				      for v := range list.All() {
				          fmt.Printf("节点值: %d\n", v)
				          
				          if v == 20 {
				              fmt.Println("  -> 找到 20 了，演示一下提前退出")
				              break
				          }
				      }
				  }
				  ```
				- **before：**
					- 调用者为了遍历它，必须知道链表的底层细节（`Next` 指针），代码很容易写错。
					- ```go
					  // ❌ 以前的麻烦写法：暴露了底层细节
					  for curr := list.Head; curr != nil; curr = curr.Next {
					      fmt.Println(curr.Val)
					  }
					  ```
				- **after：**
					- 链表变成了“黑盒”，调用者只需要关心数据。
					- ```go
					  // ✅ 现在的优雅写法：像遍历切片一样自然
					  for v := range list.All() {
					      fmt.Println(v)
					  }
					  ```
		- **坑点一：Value 是拷贝**
			- 在 `for i, v := range slice` 中，变量 `v` 是切片中元素的副本。
			- 修改 `v` 不会改变原切片！如果想修改原切片，必须使用索引 `slice[i]`。
			- ```go
			  arr := []int{1, 2}
			  for _, v := range arr {
			      v = 100 // ❌ 无效！这只是改了副本
			  }
			  
			  for i := range arr {
			      arr[i] = 100 // ✅ 有效！修改了原数据
			  }
			  ```
		- **坑点二：range 表达式只计算一次**
			- `range` 后面的对象（数组/切片等）在循环开始前会先被“快照”一次（对于数组是拷贝，对于切片是拷贝长度信息）。
			- ```go
			  nums := []int{1, 2, 3}
			  for i, v := range nums {
			      if i == 0 {
			          nums = append(nums, 4) // 在循环里追加元素
			      }
			      fmt.Println(v)
			  }
			  // 输出：1, 2, 3
			  // 即使中间追加了 4，循环也不会输出 4，因为 range 在开始时就认定了长度是 3。
			  ```
	- **变体 4：do this N times**
	  collapsed:: true
		- ```go
		  for i := range 3 {
		      fmt.Println("range", i)
		  }
		  ```
		-
-