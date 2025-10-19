- Go 里没有像三目运算符那样的简写，条件分支判断都是通过 `if` / `else` 或 `switch` 来实现。
- ## if 语句
	- **不需要圆括号，但大括号是必须的。**
	- **基本语法：**
		- ```go
		  if 条件 {
		      // 条件成立时执行
		  }
		  
		  if 条件 {
		      // 条件成立
		  } else if 其他条件 {
		      // 其他条件成立
		  } else {
		      // 都不成立
		  }
		  ```
	- **`if` 之前可带短语句：**
		- 允许你在 `if` 条件判断之前，先执行一个“初始化语句”，通常是一个**短变量声明**，这个变量只在 `if` 和 `else` 块中可见，一旦 `if-else` 块结束，变量就会被销毁。
		- **这种设计的好处是：**
			- **简洁**：将变量的“创建”和“使用”紧密地放在一起。
			- **安全**：防止变量“泄漏”到外部作用域，避免了不必要的命名冲突和 bug。当你读完 `if` 块，你就可以彻底忘记这个变量。
		- **语法：**
			- ```go
			  if [short statement]; [condition] { ... }
			  ```
		- **示例：**
			- ```go
			  if x := 10; x > 5 {
			      fmt.Println("大于 5")
			  }
			  ```
- ## switch 语句
	- **基本语法：**
		- ```go
		  switch 表达式 {
		  case 值1:
		      // 条件1
		  case 值2:
		      // 条件2
		  default:
		      // 其他情况
		  }
		  ```
	- **`switch` 也可以像 `if` 一样，带一个短语句**
		- ```go
		  switch os := runtime.GOOS; os {
		  case "darwin":
		      fmt.Println("macOS.")
		  case "linux":
		      fmt.Println("Linux.")
		  default:
		      fmt.Printf("%s.\n", os)
		  }
		  ```
	- **自动 `break`：**
		- 在 C 或 Java 中，`switch` 语句具有“代码穿透（fallthrough）”特性——当某个 `case` 被匹配后，程序会继续执行下面的所有 `case`，直到遇到 `break`。这也是初学者常犯的错误之一。
		- 而在 Go 语言中，每个 `case` 结尾都会**自动添加 `break`**，匹配并执行完一个分支后会**立即退出** `switch`。
		- 若你确实需要像 C 那样的穿透效果，可以使用 `fallthrough` 关键字显式实现。
			- ```go
			  package main
			  
			  import "fmt"
			  
			  func main() {
			      num := 2
			  
			      switch num {
			      case 1:
			          fmt.Println("One")
			      case 2:
			          fmt.Println("Two")
			          fallthrough // 显式穿透到下一个 case
			      case 3:
			          fmt.Println("Three")
			      default:
			          fmt.Println("Other")
			      }
			  }
			  ```
	- **无条件的 `switch`**
		- **语法**：`switch { ... }`
		- **行为**：`switch` 会从上到下查找**第一个 `case` 表达式为 `true` 的分支**，执行它，然后退出。
		- ```go
		  t := time.Now() // 假设现在是下午 3 点 (t.Hour() == 15)
		  switch { // <-- 无条件 switch，等价于 switch true
		  case t.Hour() < 12: // 检查: true == (15 < 12) ? (false)
		      fmt.Println("Good morning!")
		  
		  case t.Hour() < 17: // 检查: true == (15 < 17) ? (true)
		      fmt.Println("Good afternoon.") // <-- 匹配成功！执行此行
		      // 自动 break，退出 switch
		  
		  default: // 这个分支根本不会被检查
		      fmt.Println("Good evening.")
		  }
		  ```
		- **比 `if-else` 链更好：**
			- **为什么更好？**：这种 `switch { ... }` 结构比 `if-else` 链更易读，因为你不需要重复写 `else if`，所有的条件都整齐地对齐了。
			- ```go
			  if t.Hour() < 12 {
			      fmt.Println("Good morning!")
			  } else if t.Hour() < 17 {
			      fmt.Println("Good afternoon.")
			  } else {
			      fmt.Println("Good evening.")
			  }
			  ```
			- 无条件 `switch` 的形式在视觉上更清晰、更扁平。
	- `case`
		- `case` 是用来定义一个“分支”的标签。
		- **case 的工作机制：**
			- **比较**：
				- `switch` 会用它的“条件表达式”*从上到下*地与每一个 `case` 后面跟着的“值”进行**比较**。
				- 对于 `switch x { case y: }`，Go 检查 `x == y`。
				- 对于 `switch { case z: }`，Go 检查 `z == true`。
			- **顺序**：`case` 严格按照**从上到下**的顺序被求值和检查。
			- **短路：**当 `switch` 语句中的某个 `case` 被匹配后，Go 会立即执行该分支的代码，并**立刻退出**整个 `switch` 结构，而**不会继续检查或计算**后续的 `case`。
		- **`case` 的后面可以跟：**
			- **字面量：**字符串、整数、浮点数、布尔值。
			- **变量和表达式：**
				- **`case` 的值不必是常量，它可以是任何在运行时才能确定值的表达式**。
				- `switch` 会计算 `case` 后面的表达式，然后用这个**计算结果**去和 `switch` 的条件表达式进行比较。
				- ```go
				  func getStatus() int {
				      return 1
				  }
				  status := 1
				  
				  switch status {
				  case getStatus(): // <-- case 可以是一个函数调用
				      fmt.Println("Status matches function result")
				  case 1 * 2: // <-- case 可以是一个计算
				      fmt.Println("Status is two")
				  default:
				      fmt.Println("Default")
				  }
				  ```
			- **多个值：**
				- 如果你希望**多个值**都执行同一段逻辑，你不需要为它们分别写 `case`。Go 允许你用逗号 (`,`) 把它们列在同一个 `case` 后面。
				- ```go
				  switch role {
				  case "admin", "owner": // 只要 role 是 "admin" 或 "owner"
				      fmt.Println("Full access granted")
				  case "guest", "viewer": // 只要 role 是 "guest" 或 "viewer"
				      fmt.Println("Read-only access")
				  default:
				      fmt.Println("Access denied")
				  }
				  
				  switch num {
				  case 1, 3, 5, 7, 9:
				      fmt.Println("Odd")
				  case 2, 4, 6, 8:
				      fmt.Println("Even")
				  }
				  ```
			-
-