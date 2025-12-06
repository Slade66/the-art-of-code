基本数据类型
heading:: true
collapsed:: true
	- 基本数据类型是 Go 内置的类型，通常用于表示最简单的值。
	- 数值类型
	  heading:: true
		- **整型（Integer Types）：**
			- **有符号：**
				- **int**：平台相关的整数类型，通常为 32 位或 64 位。是整数类型字面量的默认类型。
				- **int8**：8 位有符号整数，取值范围为 -128 到 127。
				- **int16**：16 位有符号整数，取值范围为 -32,768 到 32,767。
				- **int32**：32 位有符号整数，取值范围为 -2,147,483,648 到 2,147,483,647。
				- **int64**：64 位有符号整数，取值范围为 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807。
			- **无符号：**
				- **uint**：平台相关的无符号整数，通常为 32 位或 64 位。
				- **uint8**：8 位无符号整数，取值范围为 0 到 255。
				- **uint16**：16 位无符号整数，取值范围为 0 到 65,535。
				- **uint32**：32 位无符号整数，取值范围为 0 到 4,294,967,295。
				- **uint64**：64 位无符号整数，取值范围为 0 到 18,446,744,073,709,551,615。
				- [[uintptr]]
			- **平台自适应整型：**
				- `int`, `uint`, `uintptr`
				- 这些类型的具体大小（32位还是64位）取决于你编译程序的操作系统。
				- 在 32 位系统上，它们是 32 位宽。
				- 在 64 位系统上，它们是 64 位宽。
				- **核心建议**：**除非有特殊需求（如内存优化、与底层系统交互），否则在需要整数时，应始终默认使用 `int`。**
		- **浮点型（Floating-point Types）：**
			- 用于表示带有小数的数字。
			- **float32**：32 位浮点数。
			- **float64**：64 位浮点数，提供的精度更高，是浮点类型字面量的默认类型。
		- **复数类型（Complex Types）：**
			- 用于科学和工程计算，表示复数（包含实部和虚部）。在常规业务开发中很少使用。
			- **complex64**：包含两个 32 位浮点数的复数。
			- **complex128**：包含两个 64 位浮点数的复数。是复数类型字面量的默认类型。
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
		- [[The Go Programming Language/字符串]]
	- **字节类型（Byte Type）：**
		- **byte**：是 `uint8` 的别名。表示一个 8 位无符号整数，常用于处理 ASCII 字符或字节流。
	- **字符类型（Character Type）：**
		- **rune**：
			- Go 的字符类型是 `rune`，它是 `int32` 的别名，用来表示一个 Unicode 码点（即字符对应的数字编号），占用 32 位（4 字节）。
			- `rune` 类型的字面量使用单引号括起来。
			- **`byte` vs `rune` 的区别：**`byte` 是一个 8 位无符号整数，只能表示 0–255 的数值，这足以覆盖常见的 ASCII 字符，如 `'a'`、`'b'`、`'c'`。而 Unicode 标准包含超过一百万个码点，UTF-8 则使用 1 到 4 个字节的可变长度序列来编码这些码点。`rune` 是 Go 中的 `int32` 类型，大小足以表示任何 Unicode 码点，因此可以表示世界上任何语言的字符，比如 `'中'` 或 emoji `'😊'`，从而弥补了 `byte` 在表示非 ASCII 字符时的局限性。
- 复合数据类型
  heading:: true
  collapsed:: true
	- 复合数据类型是由基本数据类型组合而成的更复杂的数据结构。
	- **数组（Arrays）：**
		- [[The Go Programming Language/数组]]
	- **切片（Slices）：**
		- [[The Go Programming Language/slice]]
	- **字典（Maps）：**
		- [[The Go Programming Language/map]]
	- **结构体（Structs）：**
		- [[The Go Programming Language/struct]]
	- **接口（Interfaces）：**
		- [[The Go Programming Language/接口]]
	- **函数类型（Function Types）：**
		- [[The Go Programming Language/函数]]
	- **指针类型（Pointer Types）：**
		- [[The Go Programming Language/指针]]
	- **通道类型（Channel Types）：**
- 类型转换
  heading:: true
  collapsed:: true
	- 不同类型的变量不能直接进行数学或比较运算。
	- 类型转换用于将一个值从一种类型转换为另一种类型。
	- 在 Go 语言中，类型转换必须是显式的，Go 编译器不会自动进行隐式类型转换。你必须明确地告诉编译器你要将一个值从一种类型转换成另一种类型。
	- **语法：**`T(V)`
		- 目标类型(待转换的值)
		- **T**: 你想要转换成的目标类型。
		- **V**: 你要转换的值或变量。
		- 这个表达式会创建一个类型为 `T` 的新值，其值来源于 `v`。
		- ```go
		  int(1.4) // 结果为 1
		  ```
	- **注意：**
		- 浮点数转换为整数时，小数部分会被舍弃，不进行四舍五入。
		- 数值与字符串之间的转换通常需要使用 `strconv` 包。
			- `string()` 将整数视为 Unicode 码点（rune），`string(97)` 的结果是 `'a'` 而不是 `"97"`。
			- 若要将数字转为字符串形式，应使用 `strconv.Itoa(97)`。
		- 大整数转换为小整数时，丢弃多出来的比特，但不会报错。
		- 大浮点数转换为小浮点数时，可能会丢失精度，但不会报错。
- **无符号整数溢出后会发生回绕**
  collapsed:: true
	- ```go
	  var u uint8 = 255
	  u = u + 1
	  fmt.Println(u)
	  ```
	- 这段代码执行后，变量 `u` 的值将变为 0。
	- **底层发生了什么？**
		- `uint8` 的取值范围：最小是 $0$，最大是 $2^8 - 1 = 255$。
		- 当 `u = 255` 时，8 个位全部填满为 1：$1111\ 1111_2$
		- 根据二进制加法规则（逢二进一），由于每一位都是 1，进位（Carry）会一直向左传递，直到第 9 位：${\color{Red}1}\ 0000\ 0000$
		- **截断（Overflow）：**因为 uint8 只能存储 8 位，计算机没有空间存储最左边红色的那个 1（第 9 位）。这个进位位会被直接丢弃。
		- 剩下的就是：$0000\ 0000_2$
		- 这就是十进制的 0。
	- **注意：**
		- **不会报错**：在 Go 语言的标准编译运行中，这不会导致程序崩溃（Panic），而是静默地回绕到 0。
		- 这种静默的溢出可能会导致逻辑 Bug。
- **有符号整数溢出后，最大的正数瞬间变成了最小的负数。**
  collapsed:: true
	- ```go
	  var u int8 = 127
	  u = u + 1
	  fmt.Println(u)
	  ```
	- 这段代码执行后，变量 `u` 的值将变为 -128。
	- **底层原理：**
		- 计算机使用补码来存储有符号整数。
		- **最高位被用作符号位：**
			- 0 开头代表正数。
			- 1 开头代表负数。
		- 127 是 `int8` 能表示的最大正数：${\color{Green}0}111\ 1111_2$
		- +1 后，根据二进制加法，进位一直向左传递，最终改变了最高位：${\color{Red}1}000\ 0000_2$
		- 计算机看到这个结果时，发现它的最高位是 1，因此认为它是一个负数。
		- 在补码规则中，$1000\ 0000$ 被定义为该类型能表示的最小负数，即 -128。
-