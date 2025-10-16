- **为什么需要泛型？**
	- 在没有泛型之前，如果你想写一个函数来处理不同类型的数据，通常有两种方法：
		- **方法一：为每种类型写一个函数（代码重复）**
			- ```go
			  // int 版本的 Sum
			  func SumInts(nums []int) int {
			      var total int
			      for _, num := range nums {
			          total += num
			      }
			      return total
			  }
			  
			  // float64 版本的 Sum
			  func SumFloats(nums []float64) float64 {
			      var total float64
			      for _, num := range nums {
			          total += num
			      }
			      return total
			  }
			  
			  // 用起来是这样
			  SumInts([]int{1, 2, 3})
			  SumFloats([]float64{1.1, 2.2, 3.3})
			  ```
			- 逻辑完全一样，仅仅因为类型不同就要写两遍。如果再来一个 `int32`、`float32` 类型呢？代码会变得非常冗余。
		- **方法二：使用空接口 `interface{}`（牺牲类型安全）**
			- `interface{}` 可以表示“任何类型”，但需要对每个元素进行类型断言，这不仅写起来繁琐，而且容易在运行时出错。
	- **泛型要解决的是代码复用和类型安全的问题：**编写一份代码，可以安全、高效地处理多种类型。
- **泛型的语法：**
	- **类型形参（Type Parameters）：**
		- 类型形参的声明位于函数名和参数列表之间，用方括号 `[]` 包裹。它定义一个或多个类型变量，这些类型变量可以在函数签名和函数体中使用。
		- ```go
		  //                                 [T any] 就是类型形参列表
		  //                                  │  └─ 类型约束 (Constraint)
		  //                                  └── 类型形参 (Parameter)
		  func Sum[T any](nums []T) T {
		      // ... 函数体
		  }
		  ```
		- `[T any]`：类型形参列表。
		- `T`：这是类型形参，它就像一个类型的占位符。在函数被调用时，Go 会根据传入的实参推断出 `T` 的具体类型。
		- `any`: 这是对 `T` 的类型约束，`any` 是 Go 内置的一个约束，代表“可以是任何类型”。
	- **类型约束（Type Constraints）：**
		- **例子：**
			- 如果我们直接这样写，编译器会报错：
			- ```go
			  func Sum[T any](nums []T) T {
			      var total T
			      for _, num := range nums {
			          total += num // 编译错误：invalid operation: total += num (T is not a numeric type)
			      }
			      return total
			  }
			  ```
			- 编译器说：你凭什么认为 `T` 类型可以做 `+` 运算？
		- **类型约束的作用：**它规定了类型形参 `T` 必须满足哪些条件。约束通常是一个接口类型。
		- **常用的内置约束：**
			- `any`：允许任何类型，是 `interface{}` 的别名。
			- `comparable`：允许所有支持 `==` 和 `!=` 比较的类型。包括数字、字符串、指针、数组等，但不包括 `slice`、`map` 和 `func`。
-