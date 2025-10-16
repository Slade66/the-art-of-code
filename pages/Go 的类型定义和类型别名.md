- **`type` 关键字：**
	- **作用**：`type` 是 Go 语言中用于声明一个新类型的关键字。这是所有自定义类型的起点。
	- Go 语言使用 `type` 关键字来定义新类型或类型别名，两者在使用上有显著区别。
- 类型定义
  heading:: true
	- 类型定义会创建一个全新且独立的类型。即使底层类型相同，新类型与原始类型也被视为不同，不能直接混用。
	- **语法：**
		- ```go
		  type 新类型名 基础类型
		  ```
	- **示例：**
		- 假设我们要处理温度。直接使用 `float64` 可能会导致摄氏度和华氏度混淆。我们可以定义新的类型来增加安全性。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  // 定义一个新类型 Celsius，底层类型为 float64
		  type Celsius float64
		  
		  // 定义另一个新类型 Fahrenheit，底层同为 float64
		  type Fahrenheit float64
		  
		  func main() {
		      var c Celsius = 25.0
		      var f Fahrenheit = 77.0
		  
		      fmt.Printf("当前摄氏温度: %v\n", c)
		      fmt.Printf("当前华氏温度: %v\n", f)
		  
		      // 下述代码会报错，因为 Celsius 和 Fahrenheit 是不同的类型
		      // c = f // cannot use f (type Fahrenheit) as type Celsius in assignment
		  
		      // 必须进行显式类型转换
		      c = Celsius(f)
		      fmt.Printf("转换后的摄氏温度: %v\n", c)
		  }
		  ```
- 类型别名
  heading:: true
	- 类型别名不会创建新类型，它只是为现有类型起了一个新名字。别名与原类型完全等价，可以互相赋值和混用。
	- **语法：**
		- ```go
		  type 别名 = 原类型
		  ```
		- **注意：**类型别名语法中包含一个 `=` 号，区别于类型定义。
	- **示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      // 为 string 类型创建别名 Text
		      type Text = string
		  
		      var s string = "Hello"
		      var t Text = "World"
		  
		      // 因为 Text 与 string 等价，可以互相赋值
		      s = t
		      t = s
		  
		      fmt.Println(s, t) // 输出 "World World"
		  }
		  ```
-