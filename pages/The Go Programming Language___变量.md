- **变量的声明：**
	- 使用 `var` 关键字显式声明变量（可以是一个，也可以是多个），后跟变量名和变量中要保存的值的类型。
	- 仅声明，无初始值。必须提供类型，变量会被赋予其类型的零值。
	- **语法：**
		- ```go
		  var 变量名 值类型
		  ```
	- **示例：**
		- ```go
		  var a int // 一次声明一个变量
		  var x, y int // 一次声明多个变量，如果类型相同，可在最后统一指定类型，前面的变量可省略类型声明
		  ```
		- **声明一个“变量列表”（“分组”声明）：**
			- 当需要一次性声明多个变量时，可以使用圆括号 `()` 将它们组合起来，这和 `import` 语句的分组方式非常相似，能让代码更整洁（少写许多 var 关键字）。
			- ```go
			  var (
			      c      bool
			      python bool
			      java   bool
			  )
			  ```
- **变量的赋值：**
	- ```go
	  // 单变量赋值
	  var a int
	  a = 10
	  
	  // 多变量赋值
	  var a, b int
	  a, b = 1, 2
	  ```
- **变量的初始化：**
	- **声明并初始化 (显式类型)：**
		- 如果已知变量的值，可以在声明的同时进行初始化。
		- ```go
		  var a int = 4
		  ```
	- **声明并初始化 (类型推导)：**
		- 如果你在声明变量的同时提供了初始值，就可以省略显式的类型声明；变量会自动采用初始值的类型。
		- Go 编译器会根据所赋的值的类型自动推断出变量的类型，因此在声明并初始化变量时可以省略类型；但如果未赋初始值，Go 无法确定变量将保存的数据类型，此时必须显式指定类型。
		- **示例：**
			- ```go
			  var b = 5 // 可以省略类型声明
			  ```
	- **短变量声明：**
		- 短变量声明是在函数内部快速声明并初始化变量的语法糖。
		- 还可以进一步使用短变量声明来简化变量的声明和初始化语句，这种语法糖可以同时省略 `var` 关键字和变量类型，只需写出变量名和初始值。
		- `:=` 是 `var` 关键字和 `=` 赋值操作符的简写形式。
		- 它能做两件事：
			- 声明一个新变量。
			- 初始化变量，并自动进行类型推导。
		- **示例：**
			- ```go
			  a := 10 // 短变量声明方式
			  // 相当于：
			  var a int = 10 // 传统 var 方式
			  ```
		- **注意：**
			- `:=` 的作用是声明并初始化新变量，不能用于重新声明已有的变量。
				- ```go
				  a := 1
				  a := 2 // 报错：no new variables on left side of :=
				  ```
			- **只能在函数内部使用**：简短声明语法只能在函数体内使用，不能在函数外部使用。
			- **短变量声明（`:=`）必须包含至少一个新变量。**
				- 在使用 `:=` 时，只要引入了至少一个新变量，就可以和旧变量一起使用：
					- 对新变量，会进行声明并初始化；
					- 对旧变量，只会更新其值，不会重新声明。
				- 如果 `:=` 左侧没有新变量，应该使用普通的 `=`。
				- ```go
				  a := 1
				  a, b := 2, 3 // a 是赋值，b 是声明并初始化
				  ```
- **变量的零值：**
  id:: 68f05b45-cb59-4906-8647-02713479d0c4
	- 零值是一个变量的默认值。如果变量只声明而未赋值和初始化，Go 会自动赋予其类型的零值（默认值）。
	- 你可以把它想象成一个“出厂设置”。如果在声明变量时你没有告诉 Go 它应该是什么，Go 就会给它一个预设好的、干净的初始状态，而不是留下一块存有“垃圾”数据的内存。
	- 不同类型有不同的零值：
		- 所有数字类型的零值是：`0`
		- 字符串类型的零值是：`""`
		- 布尔类型的零值是：`false`
	- **代码示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		  	// 数值类型的零值
		  	var i int
		  	var i8 int8
		  	var i16 int16
		  	var i32 int32
		  	var i64 int64
		  	var ui uint
		  	var ui8 uint8
		  	var ui16 uint16
		  	var ui32 uint32
		  	var ui64 uint64
		  	var uintptr uintptr
		  	
		  	// 浮点数类型的零值
		  	var f32 float32
		  	var f64 float64
		  	
		  	// 复数类型的零值
		  	var c64 complex64
		  	var c128 complex128
		  	
		  	// 布尔类型的零值
		  	var b bool
		  	
		  	// 字符串类型的零值
		  	var s string
		  	
		  	// 字节和rune类型的零值
		  	var by byte // byte 是 uint8 的别名
		  	var r rune  // rune 是 int32 的别名
		  	
		  	// 指针类型的零值
		  	var ptr *int
		  	
		  	// 切片类型的零值
		  	var slice []int
		  	
		  	// 映射类型的零值
		  	var m map[string]int
		  	
		  	// 通道类型的零值
		  	var ch chan int
		  	
		  	// 函数类型的零值
		  	var fn func()
		  	
		  	// 接口类型的零值
		  	var iface interface{}
		  	
		  	// 结构体类型的零值
		  	type Person struct {
		  		Name string
		  		Age  int
		  	}
		  	var person Person
		  	
		  	// 数组类型的零值
		  	var arr [3]int
		  	
		  	fmt.Println("=== 数值类型的零值 ===")
		  	fmt.Printf("int: %v\n", i)
		  	fmt.Printf("int8: %v\n", i8)
		  	fmt.Printf("int16: %v\n", i16)
		  	fmt.Printf("int32: %v\n", i32)
		  	fmt.Printf("int64: %v\n", i64)
		  	fmt.Printf("uint: %v\n", ui)
		  	fmt.Printf("uint8: %v\n", ui8)
		  	fmt.Printf("uint16: %v\n", ui16)
		  	fmt.Printf("uint32: %v\n", ui32)
		  	fmt.Printf("uint64: %v\n", ui64)
		  	fmt.Printf("uintptr: %v\n", uintptr)
		  	
		  	fmt.Println("\n=== 浮点数类型的零值 ===")
		  	fmt.Printf("float32: %v\n", f32)
		  	fmt.Printf("float64: %v\n", f64)
		  	
		  	fmt.Println("\n=== 复数类型的零值 ===")
		  	fmt.Printf("complex64: %v\n", c64)
		  	fmt.Printf("complex128: %v\n", c128)
		  	
		  	fmt.Println("\n=== 布尔类型的零值 ===")
		  	fmt.Printf("bool: %v\n", b)
		  	
		  	fmt.Println("\n=== 字符串类型的零值 ===")
		  	fmt.Printf("string: %q\n", s)
		  	
		  	fmt.Println("\n=== 字节和Rune类型的零值 ===")
		  	fmt.Printf("byte: %v\n", by)
		  	fmt.Printf("rune: %v\n", r)
		  	
		  	fmt.Println("\n=== 指针类型的零值 ===")
		  	fmt.Printf("*int: %v\n", ptr)
		  	
		  	fmt.Println("\n=== 切片类型的零值 ===")
		  	fmt.Printf("[]int: %v, len=%d, cap=%d\n", slice, len(slice), cap(slice))
		  	
		  	fmt.Println("\n=== 映射类型的零值 ===")
		  	fmt.Printf("map[string]int: %v\n", m)
		  	
		  	fmt.Println("\n=== 通道类型的零值 ===")
		  	fmt.Printf("chan int: %v\n", ch)
		  	
		  	fmt.Println("\n=== 函数类型的零值 ===")
		  	fmt.Printf("func(): %v\n", fn)
		  	
		  	fmt.Println("\n=== 接口类型的零值 ===")
		  	fmt.Printf("interface{}: %v\n", iface)
		  	
		  	fmt.Println("\n=== 结构体类型的零值 ===")
		  	fmt.Printf("Person: %+v\n", person)
		  	
		  	fmt.Println("\n=== 数组类型的零值 ===")
		  	fmt.Printf("[3]int: %v\n", arr)
		  }
		  ```
		- ```text
		  === 数值类型的零值 ===
		  int: 0
		  int8: 0
		  int16: 0
		  int32: 0
		  int64: 0
		  uint: 0
		  uint8: 0
		  uint16: 0
		  uint32: 0
		  uint64: 0
		  uintptr: 0
		  
		  === 浮点数类型的零值 ===
		  float32: 0
		  float64: 0
		  
		  === 复数类型的零值 ===
		  complex64: (0+0i)
		  complex128: (0+0i)
		  
		  === 布尔类型的零值 ===
		  bool: false
		  
		  === 字符串类型的零值 ===
		  string: "" (空字符串)
		  
		  === 字节和Rune类型的零值 ===
		  byte: 0
		  rune: 0
		  
		  === 指针类型的零值 ===
		  *int: <nil> (nil)
		  
		  === 切片类型的零值 ===
		  []int: [] (nil), len=0, cap=0
		  
		  === 映射类型的零值 ===
		  map[string]int: map[] (nil)
		  
		  === 通道类型的零值 ===
		  chan int: <nil> (nil)
		  
		  === 函数类型的零值 ===
		  func(): <nil> (nil)
		  
		  === 接口类型的零值 ===
		  interface{}: <nil> (nil)
		  
		  === 结构体类型的零值 ===
		  Person: {Name: Age:0} (所有字段都是零值)
		  
		  === 数组类型的零值 ===
		  [3]int: [0 0 0] (所有元素都是零值)
		  ```
- **变量的命名规则：**
	- 变量名只能包含字母、数字和下划线，大小写敏感，且必须以字母开头。
	- 首字母大写表示导出（可被其他包访问），首字母小写表示不导出（仅在包内可见）。
	- 变量名通常以小写字母开头，若由多个单词组成，除第一个单词外，其余单词的首字母需大写，单词之间不使用空格或下划线，而是直接连接在一起。
	- **不能用于变量名的名称：**
		- **关键字：**
			- 这些是 Go 语言语法的一部分，编译器通过它们识别控制结构、函数定义等语句。不能用作变量名，否则会导致编译错误。
			- ```go
			  // 关键字（绝对不能用）
			  break    default    func     interface  select
			  case     defer      go       map        struct
			  chan     else       goto     package    switch
			  const    fallthrough if      range      type
			  continue for        import   return     var
			  ```
		- **内置的标识符：**
			- 不会报错，但不推荐这么做。
			- **内置类型名**（如 `int`、`string`）：可以作为变量名，但会遮盖类型，导致后续无法定义该类型的变量或进行类型转换。
			- **内置函数名**（如 `append`、`len`）：可以作为变量名，但会遮盖对应的内置函数，后续无法再调用该函数。
			- **标准库包名**（如 `fmt`、`os`、`time`）：可以作为变量名使用，但会遮盖你导入的包，影响对包内函数的正常调用。
- **变量的遮盖问题：**
	- 在内层作用域中声明了一个与外层作用域同名的变量，从而“遮盖”了外层变量，导致外层变量在该作用域中不可见或无法使用。这种情况不会报错，但可能会引起代码逻辑混乱或难以发现的 bug。
- **变量的作用域：**
	- `var` 语句可以出现在包级别或函数级别。
	- **作用域 (Scope)** 指的是一个变量在代码中可以被访问的范围。`var` 语句的位置决定了它的作用域。
	- **包级别：**
		- **位置**：在**任何函数之外**声明。
		- **可见性**：对**同一个包内的所有函数**都是可见的，可以被它们共享和访问。
		- **生命周期**：从程序开始运行到程序结束。
	- **函数级别：**
		- **位置**：在**函数体内部**声明。
		- **可见性**：**仅在该函数内部**可见和使用。
		- **生命周期**：从进入函数开始，到函数执行完毕退出时销毁。
- **`_` 空白标识符**
	- 用于丢弃不需要使用的变量值。
	- `_`（空白标识符）不被视为“新变量”。
		- `a := 1; a, _ := 2, 3`
			- `:=` 的左侧只有 `a` 这一个已声明的变量，导致“没有新变量”的错误。