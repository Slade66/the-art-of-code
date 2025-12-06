## 有什么用？
	- `type` 是 Go 语言中用于声明一个新类型或为现有类型起别名的关键字。`type` 不仅用于定义结构体、接口和函数，还允许你基于现有类型创建新的自定义类型。
- 类型定义
  heading:: true
	- **语法：**
		- ```go
		  type 新类型名 基础类型
		  ```
	- 类型定义会创建一个全新的类型。即使底层类型相同，新类型与原始类型也被视为不同，必须显式类型转换。
		- ```go
		  var a int = 10
		  var b MyInt = a
		  ```
		- Go 是强类型语言，`MyInt` 和 `int` 被视为不同类型，必须使用 `MyInt(a)` 进行显式转换。
	- **用法：**
		- **增强语义：**`type Email string` 比单纯的 `string` 更能表达变量的含义。
		- **绑定方法**：你可以为 `Duration` 定义特有的方法（如 `d.Hours()`），但不能直接给内置类型 `int64` 定义方法。
		- **结构体定义：**用于定义复杂的复合数据类型，将多个字段组合在一起。
			- ```go
			  type User struct {
			      ID       int
			      Name     string
			      IsActive bool
			  }
			  ```
		- **接口定义：**用于定义行为契约。接口定义了一组方法签名，任何实现了这些方法的类型都满足该接口。
			- ```go
			  type Reader interface {
			      Read(p []byte) (n int, err error)
			  }
			  ```
		- **函数类型：**你可以使用 `type` 给特定的函数签名命名。
			- ```go
			  type HandlerFunc func(ParameterTypes) ReturnTypes
			  
			  // 定义一个计算函数的类型
			  type MathOp func(int, int) int
			  
			  func add(a, b int) int { return a + b }
			  
			  func main() {
			      var op MathOp = add
			      println(op(1, 2))
			  }
			  ```
- 类型别名
  heading:: true
	- 类型别名不会创建新类型，它只是为现有类型起了一个新名字。别名与原类型完全等价，可以互相赋值而不需要转换。
		- ```go
		  package main
		  
		  func main() {
		  	type str = string
		  
		  	var a string = "haha"
		  	var b str = "xixi"
		  	b = a // 可以赋值，因为是相同的类型
		  	println(b)
		  }
		  ```
	- **语法：**
		- ```go
		  type 别名 = 原类型
		  ```
		- **注意：**类型别名语法中包含一个 `=` 号，区别于类型定义。
-