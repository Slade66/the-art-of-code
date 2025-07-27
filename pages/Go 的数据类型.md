基本数据类型
heading:: true
	- 基本数据类型是 Go 内置的类型，通常用于表示最简单的值。
	- 数值类型
	  heading:: true
		- **整型（Integer Types）：**
			- **无符号：**
				- **int**：平台相关的整数类型，通常为 32 位或 64 位。
				- **int8**：8 位有符号整数，取值范围为 -128 到 127。
				- **int16**：16 位有符号整数，取值范围为 -32,768 到 32,767。
				- **int32**：32 位有符号整数，取值范围为 -2,147,483,648 到 2,147,483,647。
				- **int64**：64 位有符号整数，取值范围为 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807。
			- **有符号：**
				- **uint**：平台相关的无符号整数，通常为 32 位或 64 位。
				- **uint8**：8 位无符号整数，取值范围为 0 到 255。
				- **uint16**：16 位无符号整数，取值范围为 0 到 65,535。
				- **uint32**：32 位无符号整数，取值范围为 0 到 4,294,967,295。
				- **uint64**：64 位无符号整数，取值范围为 0 到 18,446,744,073,709,551,615。
		- **浮点型（Floating-point Types）：**
			- **float32**：32 位浮点数，符合 IEEE 754 标准。
			- **float64**：64 位浮点数，符合 IEEE 754 标准。
		- **复数类型（Complex Types）：**
			- **complex64**：包含两个 32 位浮点数的复数。
			- **complex128**：包含两个 64 位浮点数的复数。
		- **代码示例：**
			- ```go
			  package main
			  
			  import "fmt"
			  
			  func main() {
			  
			  	// Go 数值类型测试
			  	var i int = 42
			  	var i8 int8 = -8
			  	var i16 int16 = 1600
			  	var i32 int32 = 32000
			  	var i64 int64 = 6400000000
			  	var ui uint = 42
			  	var ui8 uint8 = 8
			  	var ui16 uint16 = 1600
			  	var ui32 uint32 = 32000
			  	var ui64 uint64 = 6400000000
			  	var f32 float32 = 3.14
			  	var f64 float64 = 2.718281828
			  	var c64 complex64 = 1 + 2i
			  	var c128 complex128 = 2 + 3i
			  
			  	fmt.Println("int:", i)
			  	fmt.Println("int8:", i8)
			  	fmt.Println("int16:", i16)
			  	fmt.Println("int32:", i32)
			  	fmt.Println("int64:", i64)
			  	fmt.Println("uint:", ui)
			  	fmt.Println("uint8:", ui8)
			  	fmt.Println("uint16:", ui16)
			  	fmt.Println("uint32:", ui32)
			  	fmt.Println("uint64:", ui64)
			  	fmt.Println("float32:", f32)
			  	fmt.Println("float64:", f64)
			  	fmt.Println("complex64:", c64)
			  	fmt.Println("complex128:", c128)
			  }
			  ```
	- **布尔类型（Boolean Type）：**
		- **bool**：布尔类型，用于表示 `true` 或 `false`。
	- **字符串类型（String Type）：**
		- [[Go 的 string]]
	- **字符类型（Character Type）：**
		- **rune**：
			- Go 的字符类型是 `rune`，它是 `int32` 的别名，表示一个 Unicode 码点（即字符对应的数字编号），占用 32 位（4 字节）。
			- `rune` 类型的字面量使用单引号括起来。
- 复合数据类型
  heading:: true
	- 复合数据类型是由基本数据类型组合而成的更复杂的数据结构。
	- **数组（Arrays）：**
		- [[Go 的数组]]
	- **切片（Slices）：**
		- [[Go 的切片]]
	- **字典（Maps）：**
		- [[Go 的 map]]
	- **结构体（Structs）：**
		- [[Go 的 struct]]
	- **接口（Interfaces）：**
		- [[Go 的接口]]
	- **函数类型（Function Types）：**
		- [[Go 的函数]]
	- **指针类型（Pointer Types）：**
		- [[Go 的指针]]
- 类型转换
  heading:: true
	- 类型转换用于将一个值从一种类型转换为另一种类型。由于 Go 不会自动进行隐式类型转换，因此需要手动指定转换方式。
	- 不同类型的变量不能直接进行数学或比较运算。
	- ```go
	  语法：目标类型(待转换的值)
	  示例：int(1.4) // 结果为 1
	  ```
	- **注意：**
		- 浮点数转换为整数时，小数部分会被舍弃，不进行四舍五入。
		- 数值与字符串之间的转换通常需要使用 `strconv` 包。
		- 大整数转换为小整数时，会截断为小类型的表示范围，超出部分会丢失，但不会报错。
		- 大浮点数转换为小浮点数时，可能会丢失精度，但不会报错。
-