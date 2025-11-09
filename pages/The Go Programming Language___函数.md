- Go 中的函数本身也是一种类型，可以用来声明函数变量：
  collapsed:: true
	- ```go
	  func add(a, b int) int {
	      return a + b
	  }
	  
	  var f func(int, int) int = add  // 将函数赋值给变量
	  ```
- 函数名的命名规则和变量名相同。
- Go 不支持函数重载，以避免函数签名模糊的问题，推荐使用不同的函数名明确表达函数用途：
  collapsed:: true
	- ```go
	  // Java 风格（函数重载）：
	  class Printer {
	      void print(int val) { /* ... */ }
	      void print(String val) { /* ... */ }
	      void print(float val) { /* ... */ }
	  }
	  
	  // Go 风格（不同函数名）：
	  func PrintInt(val int) { /* ... */ }
	  func PrintString(val string) { /* ... */ }
	  func PrintFloat(val float64) { /* ... */ }
	  
	  ```
	- 或者使用方法，将同名函数关联到不同的接收者上，以区分不同类型的操作。
- **值传递：**
  collapsed:: true
	- 在调用带参数的函数时，Go 会将每个实参的值复制到对应的形参变量中，这被称为“值传递”。
	- 因为函数接收到的是变量的副本而非原始变量，所以在函数内部对参数的修改不会影响外部变量。
	- 如果希望函数能够直接修改原始变量的值，则需要通过传递变量的引用（内存地址），通过解引用操作，可以直接修改原变量的值。
	- 切片（slice）是一个结构体，它包含一个指向底层数组的指针。函数接收的是这个描述符的副本，但它指向的数组是同一个。因此，修改切片元素会影响外部。
	- Map 是引用类型。当 map 作为参数传递时，函数接收的是一个指向底层数据结构的指针的副本。对 map 的修改会反映到原始的 map 上。
- **声明函数的语法：**
  collapsed:: true
	- ```go
	  func 函数名(参数列表) 返回值类型 {
	      // 函数体
	  }
	  ```
	- **参数列表的情况：**
	  collapsed:: true
		- 请注意，类型信息是写在变量名之后的。
		- **零个参数**：函数不需要任何外部输入就能完成工作。
		- **单个参数：**`func greet(name string)`
		- **多个参数（不同类型）：**`func move(x int, y float64)`
		- **多个参数（相同类型，类型合并）：**`func add(a, b int)`
		  collapsed:: true
			- 如果连续多个参数的类型相同，你可以省略前面参数的类型声明
		- **变长参数：**`func sum(nums ...int)`
		  collapsed:: true
			- 允许函数接收任意数量（包括 0 个）的同一类型参数，这对于处理不定数量输入的场景非常实用。
			- 变长参数在函数内部会被当作一个切片处理。
			- **注意：**
			  collapsed:: true
				- 变长参数必须是函数参数列表中的最后一个参数。
				- 使用展开语法 `变量名...`，可以将一个切片或数组展开为一个个元素传入函数。
			- **函数签名是可变参数，用 `nil` 切片展开调用它会发生什么？**
			  collapsed:: true
				- 如果一个函数签名是 `func process(nums ...int)`，使用 `var s []int = nil` 的切片来调用它，即 `process(s...)`，函数体内的 `nums` 变量会是什么？
				- 函数体内的 `nums` 变量会是一个 **`nil` 切片** (`[]int`)。它的判断结果是 `nums == nil` 为 `true`，同时其长度 (`len(nums)`) 和容量 (`cap(nums)`) 都是 $0$。Go 语言允许将 `nil` 切片通过展开操作符 (`...`) 传递给可变参数。
		- **参数为复杂类型：**
		  collapsed:: true
			- 切片类型：`func printAll(items []string)`，用于传递一个字符串列表。
			- 结构体类型：`func greetUser(u User)`，用于传递包含多个字段的自定义数据结构。
			- 函数类型：`func operate(a, b int, op func(int, int) int)`，可以将函数作为参数传入。
	- **返回值的情况：**
	  collapsed:: true
		- **无返回值：**`func sayHello()`
		- **单个返回值：**`func square(x int) int`
		- **多个返回值：**
		  collapsed:: true
			- Go 语言的函数可以返回任意数量的结果。
			- collapsed:: true
			  ```go
			  func divide(a, b float64) (float64, error)
			  ```
				- 括号里定义了函数将要返回的值的类型和数量。
				- 在这里，`divide` 函数会返回两个值，一个是 `float64` 类型，另一个是 `error` 类型。
			- 一个常见的用法是返回一个结果值和一个错误值（`error`），调用方可以通过判断错误值是否为 `nil` 来确定函数是否执行成功。
			- **注意：**
			  collapsed:: true
				- Go 要求所有声明的变量必须被使用，因此如果忽略对返回的 `err` 变量的处理，代码将无法通过编译。
				- **如何忽略不想要的返回值？**
				  collapsed:: true
					- 有时候，你可能只关心函数返回的多个值中的某一个。如果你声明了一个变量却不使用它，Go 编译器会报错。这时，你可以使用 **空白标识符 (blank identifier)** `_` 来丢弃不想要的返回值。
				- 当函数需要返回多个值时，`return` 关键字后应跟随着多个用逗号隔开的值。返回值的顺序至关重要：第一个返回值对应函数签名中的第一个返回类型，第二个对应第二个，以此类推。
			- **接收多个返回值：**
			  collapsed:: true
				- `a, b := swap("hello", "world")`
				- 当调用一个返回多个值的函数时，你需要在赋值操作符 `:=` 或 `=` 的左侧提供相应数量的变量来接收它们。
		- **命名返回值：**
		  collapsed:: true
			- Go 函数的返回值可以被命名。在函数签名中给返回值起名字，它们就会被当作在函数顶部预先定义好的变量来对待，可在函数体中直接使用这些变量。
			- collapsed:: true
			  ```go
			  func split(sum int) (x, y int) {
			      x = sum * 2 / 3
			      y = sum - x
			      return // 返回的是具名变量 x 和 y
			  }
			  ```
				- 这里的 `(x, y int)` 就是关键。它不仅声明了函数会返回两个 `int` 类型的值，还给这两个返回值分别起了名字：`x` 和 `y`。
				- **最重要的效果**：从此刻起，`x` 和 `y` 就在 `split` 函数的内部被**自动声明**为两个 `int` 类型的局部变量了，并且它们的初始值是该类型的零值 (zero value)，对于 `int` 来说就是 `0`。你无需再使用 `var x int` 或 `x := ...` 来创建它们。
				- 因为 `x` 和 `y` 已经被当作变量存在于函数的作用域内了，所以你可以直接使用它们（给它们赋值）。
				- **“裸”返回：**一个不带任何参数的 `return` 语句会直接返回这些已命名的返回值。
			- **defer + 命名返回值**
			  collapsed:: true
				- ```go
				  func test() (x int) {
				      x = 10
				      defer func() {
				          x++
				      }()
				      return 20
				  }
				  ```
				- defer 函数是在 return 语句执行后才调用的。
				- 执行 `return 20` 时：
				  collapsed:: true
					- Go 会先把命名返回值 `x` 设为 20。
					- 然后，`defer` 语句被执行，它访问并修改了 `x`（`x++`），使其变为 21。最后，函数返回 `x` 的最终值 21。
			- **defer + 非命名返回值**
			  collapsed:: true
				- ```go
				  func test() int {
				      x := 10
				      defer func() {
				          x++
				      }()
				      return x
				  }
				  ```
				- `return x` 先将 `x` 的值 (10) 复制到**临时的、匿名的返回值变量**中，`defer` 虽然执行 `x++`，但修改的是**局部变量 `x`**，不影响已存储在匿名返回值变量中的 10。
			- **优点：**
			  collapsed:: true
				- **自文档化：**当一个函数返回多个相同类型的值时，命名可以让函数签名本身变得非常清晰。
				  collapsed:: true
					- **不清晰的例子**：`func findCoords() (float64, float64, float64)` -> 返回的这三个浮点数分别是什么？经度？纬度？海拔？
					- **清晰的例子**：`func findCoords() (latitude, longitude, altitude float64)` -> 一目了然！函数签名直接告诉了调用者每个返回值的确切含义。
			- **缺点：**
			  collapsed:: true
				- **“裸”返回损害可读性**：
				  collapsed:: true
					- 在 `split` 这种只有 3 行代码的函数里，“裸”返回看起来很简洁。
					- 但想象一个有 30 行代码的函数，中间有复杂的 `if-else` 分支，可能在不同分支里修改了 `x` 和 `y` 的值。当你在函数末尾看到一个孤零零的 `return` 时，你很难立刻确定 `x` 和 `y` 的最终值是什么，必须仔细地从头到尾阅读和推演整个函数的逻辑。这极大地增加了理解和维护代码的认知负担。
			- **最佳实践：**
			  collapsed:: true
				- **可以用命名返回值来提升代码清晰度**：尤其是当函数返回多个值，且它们的类型相同时，给它们命名是一种很好的代码注释方式。
				- **谨慎使用“裸”返回**：**强烈建议只在不超过 5 行的、逻辑极其简单的函数中使用它。**
				- **最佳组合**：一个很好的实践是，**使用命名返回值来获得文档化的好处，但在函数末尾明确地返回它们**。这样既清晰又安全。
		- 返回值为复杂类型：
		  collapsed:: true
			- 返回切片：`func getList() []int`
			- 返回结构体：`func newUser(name string) User`
			- 返回函数：`func multiplier(factor int) func(int) int`
- **函数声明的位置：**
  collapsed:: true
	- 在同一个包内，函数的声明顺序和位置都无关紧要，既可以放在 `main` 函数之前，也可以放在之后。Go 编译器会先收集整个包的声明，再解析调用关系，因此即使某个函数写在 `main` 之后，也能在 `main` 中直接调用。
- **为什么 Go 语言的类型要写在变量名之后？**
  collapsed:: true
	- 你可能会问，既然大家都习惯 `int x`，为什么 Go 要“特立独行”地设计成 `x int` 呢？
	- **让我们来看一个经典的 C 语言复杂声明：**
	  collapsed:: true
		- ```c
		  int (*fp)(int, int);
		  ```
		- 要读懂这个声明，你需要从中间开始，向外扩展，像剥洋葱一样：“`fp` 是一个指针，它指向一个函数，这个函数接收两个 `int` 参数并返回一个 `int`。”  这种 “螺旋式” 的解读方式非常不直观。
	- **现在看看 Go 的方式：**
	  collapsed:: true
		- ```go
		  var fp func(int, int) int
		  ```
		- 这个声明可以非常自然地从左到右阅读：
		  collapsed:: true
			- 声明一个变量 `fp`，它的类型是 `func(int, int) int` (一个接收两个 `int` 并返回一个 `int` 的函数类型)。
	- **另一个例子：指针数组**
	  collapsed:: true
		- **C 语言**：`int *p[10];`
		  collapsed:: true
			- 解读：“`p` 是一个包含 10 个元素的数组，数组的每个元素都是一个指向 `int` 的指针。” 这也需要一些思维转换。
		- **Go 语言**：`var p [10]*int`
		  collapsed:: true
			- 解读：“声明一个变量 `p`，它的类型是 `[10]*int` (一个包含 10 个元素的数组，元素类型是 `*int`)。”
	- **总结 Go 这样设计的好处：**
	  collapsed:: true
		- **从左到右的自然阅读顺序，有助于消除歧义：**变量名总是在最前面，后面紧跟其类型描述。无论声明多么复杂，你都能清楚地知道变量名是什么、类型是什么，不会像 C 语言那样出现解析困难——C 的类型信息和返回值分布在声明的左右两侧，而 Go 则统一放在变量名的右边。
- **为什么多返回值很有用？**
  collapsed:: true
	- 多返回值最主要的应用场景是：同时返回结果和错误状态。
	- 这是 Go 语言中进行错误处理的标准模式。一个函数在执行后，通常会返回它本应产生的结果以及一个 `error` 对象。
	  collapsed:: true
		- 如果操作成功，它返回结果和 `nil` (表示没有错误)。
		- 如果操作失败，它返回一个零值（或无效值）和具体的 `error` 信息。
	- **例子：打开文件**
	  collapsed:: true
		- ```go
		  import "os"
		  
		  func main() {
		      // os.Open 函数返回一个文件对象和一个 error
		      file, err := os.Open("non-existent-file.txt")
		  
		      // 这是 Go 中最常见的代码模式：立即检查错误
		      if err != nil {
		          fmt.Println("出错了:", err)
		          return // 出错就直接返回，不继续执行
		      }
		  
		      // 如果 err 是 nil，说明文件成功打开，可以继续使用 file
		      fmt.Println("文件打开成功:", file.Name())
		      file.Close()
		  }
		  ```
		- 这种模式强制开发者显式地处理可能发生的错误，而不是像其他语言那样依赖异常 `try-catch` 机制，这使得 Go 代码通常更加稳健可靠。
- **当一个函数返回另外一个函数：**
  collapsed:: true
	- **闭包：**
	  collapsed:: true
		- 闭包是一个函数值，它引用了其函数体之外的变量。
		- 闭包允许一个函数携带和维护自己的私有状态。
		- **状态的持久化：**即使外层函数执行完并返回，闭包（也就是返回的那个匿名函数）仍然保留着对外层变量的引用。那些外部变量不会被销毁，因为它们已经被闭包“绑定”住了。只要闭包还存在（例如被某个变量引用），这些被绑定的变量就会**跟着它一起存活**。
	- **例子 1：**
	  collapsed:: true
		- ```go
		  func makeAdder(x int) func(int) int {
		      return func(y int) int {
		          return x + y
		      }
		  }
		  
		  func main() {
		      add5 := makeAdder(5)
		      // add5(10) 的返回值是？
		  }
		  ```
		- 这是一个闭包。`makeAdder(5)` 返回一个函数，这个**函数“记住”了它被创建时的环境**，即 `x = 5`。调用 `add5(10)` 时，`y = 10`，因此返回 `5 + 10`。
	- **例子 2：**
	  collapsed:: true
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func makeCounter() func() int {
		      i := 0
		      return func() int {
		          i++
		          return i
		      }
		  }
		  
		  func main() {
		      counter := makeCounter() // makeCounter() returns a function
		      counter() // i = 1
		      counter() // i = 2
		      fmt.Println(counter()) // i = 3
		  }
		  ```
		- 这个匿名函数会访问并修改其外层函数中的变量 `i`。变量 `i` 不会在 `makeCounter` 返回后被销毁，而是会一直保存在内存中，由返回的函数持续引用。
		- **闭包工厂：**像 `makeCounter` 这样的函数被称为闭包工厂，因为它会返回一个函数。每次调用 `makeCounter()`，都会创建一个**全新且独立**的 `i` 变量及其闭包环境。
		  collapsed:: true
			- 例如，再定义一个 `counter2 := makeCounter()` 时，`counter2` 也会拥有自己独立的 `i`，从 0 开始计数，和 `counter` 的闭包环境互不干扰。
- **“立即调用”的匿名函数 (IIFE)**
  collapsed:: true
	- 在定义匿名函数 `func() { ... }` 之后，立即使用 `()` 来调用它。
- **函数本身就是一种值：**
  collapsed:: true
	- 你可以将一个函数赋值给一个变量。
	- 也可以把一个函数当作参数传递给另一个函数。
	- 你可以把一个函数当成另一个函数的返回值。
	- ```go
	  package main
	  
	  import "fmt"
	  
	  // 1. 定义一个函数类型
	  type operation func(int, int) int
	  
	  // 2. 将函数赋值给变量
	  var add operation = func(a, b int) int {
	  	return a + b
	  }
	  
	  // 3. 函数作为参数
	  func calculate(a, b int, op operation) int {
	  	return op(a, b)
	  }
	  
	  // 4. 函数作为返回值
	  func getOperation(opType string) operation {
	  	switch opType {
	  	case "power":
	  		return func(a, b int) int {
	  			result := 1
	  			for i := 0; i < b; i++ {
	  				result *= a
	  			}
	  			return result
	  		}
	  	case "multiply":
	  		return func(a, b int) int {
	  			return a * b
	  		}
	  	default:
	  		return add // 返回之前定义的函数变量
	  	}
	  }
	  
	  // 5. 高阶函数示例
	  func applyToEach(numbers []int, f func(int) int) []int {
	  	result := make([]int, len(numbers))
	  	for i, v := range numbers {
	  		result[i] = f(v)
	  	}
	  	return result
	  }
	  
	  func main() {
	  	// 1. 使用已赋值的函数变量
	  	fmt.Println("5 + 3 =", add(5, 3))
	  
	  	// 2. 函数作为参数传递
	  	subtract := func(a, b int) int { return a - b }
	  	fmt.Println("10 - 4 =", calculate(10, 4, subtract))
	  
	  	// 3. 获取并调用返回的函数
	  	powerFunc := getOperation("power")
	  	fmt.Println("2 ^ 3 =", powerFunc(2, 3))
	  
	  	// 4. 匿名函数直接调用
	  	fmt.Println("10 * 2 =", func(a, b int) int {
	  		return a * b
	  	}(10, 2))
	  
	  	// 5. 高阶函数使用
	  	nums := []int{1, 2, 3, 4}
	  	doubled := applyToEach(nums, func(n int) int {
	  		return n * 2
	  	})
	  	fmt.Println("Doubled:", doubled)
	  
	  	// 6. 闭包示例
	  	makeAdder := func(x int) func(int) int {
	  		return func(y int) int {
	  			return x + y
	  		}
	  	}
	  
	  	add5 := makeAdder(5)
	  	fmt.Println("5 + 10 =", add5(10)) // 15
	  }
	  ```
- **函数有自己的类型：**
  collapsed:: true
	- 一个函数的“类型”由其参数和返回值决定。
	- **示例：**
	  collapsed:: true
		- ```go
		  func compute(fn func(float64, float64) float64) float64 {
		  	return fn(3, 4)
		  }
		  ```
		- `compute` 函数的参数 `fn` 的类型就是 `func(float64, float64) float64`。任何**签名匹配**的函数都可以传给它。
-