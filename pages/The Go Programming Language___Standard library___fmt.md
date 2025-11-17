- [[type Stringer interface]]
- `fmt.Errorf`
  id:: 68fdc145-4dba-4b18-83b1-ffaa52ce82a2
  collapsed:: true
	- `fmt.Errorf` 函数允许使用格式化动词来构建错误字符串。
	- **创建错误链：`%w` 动词与 `Unwrap` 方法**
		- **Go 1.13 之前的问题：**在 Go 1.13 之前，标准库在错误处理方面存在一个显著的短板：当使用 `fmt.Errorf("... %v", err)` 为一个错误添加上下文时，原始错误的类型信息会丢失，只剩下它的字符串表示 。这使得程序无法以编程方式检查错误的根本原因，开发者不得不退而求其次，依赖脆弱的字符串匹配。
		- Go 1.13 引入了错误包装（error wrapping）的官方支持，允许一个错误“包装”（wrap）另一个错误，形成一个错误链。其核心是两个新元素：`%w` 格式化动词和 `Unwrap` 方法约定。
			- **`%w` 动词**：在 `fmt.Errorf` 函数中，使用 `%w` 动词可以创建一个包装了底层错误的新的错误值。`%w` 要求其对应的参数必须是一个 `error`。
			- **`Unwrap` 方法**：
				- 如果一个错误类型包含了另一个错误，它可以实现一个名为 `Unwrap` 的方法，该方法返回其内部包含的错误。如果 `e1.Unwrap()` 返回 `e2`，我们就说 `e1` 包装了 `e2`。
				- 通过 `%w` 创建的错误会自动实现 `Unwrap` 方法。这种机制允许将错误链接在一起，形成一个“错误链”（error chain），它就像一个单向链表，记录了错误从最初发生点到当前处理点的完整传播路径 。链上的每一层都添加了新的上下文，但并未破坏原始错误，从而为后续的检查和调试保留了全部信息。
		- 为了检查错误链，Go 1.13 在 `errors` 包中引入了两个新的、至关重要的函数：`errors.Is` 和 `errors.As`。它们会自动遍历整个错误链，使开发者不必手动解包。
			- ((68fdd9fb-9803-4ee5-b5d8-3beb69d85de2))
			- ((68fdda00-ad48-4f7e-8744-4d6b7f788aaa))
	- **代码示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func findUser(id int) error {
		      if id <= 0 {
		          return fmt.Errorf("invalid user ID: %d", id)
		      }
		      //...
		      return nil
		  }
		  
		  func main() {
		  	err := findUser(0)
		  	fmt.Println(err)
		  }
		  
		  ```
- `func Printf(format string, a ...any) (n int, err error)`
  collapsed:: true
	- **作用：**`fmt.Printf` 函数的工作方式是接收一个格式字符串，其中包含占位符（称为格式说明符），然后将后面的参数按照这些占位符的指示进行格式化输出。
	- **格式说明符：**
		- `%v`：值（Value）。Go 语言中默认、最通用的格式，会以最易读的方式打印值。
		- `%T`：类型。打印变量的类型。
		- `%d`：十进制整数。
		- `%f`：浮点数（默认输出所有小数位）。
		- `%s`：字符串。
		- `%t`：布尔值。
		- `%p`：指针地址。
	- **控制精度的修饰符：**
		- **浮点数精度：** `.Nf` 表示保留 $N$ 位小数。
			- `%.2f`：保留两位小数。
			- **示例：** `fmt.Printf("Pi is approx: %.2f\n", 3.14159)` $\rightarrow$ `Pi is approx: 3.14`
		- **字符串截断：** `.Ns` 表示最多打印 $N$ 个字符。
			- **示例：** `fmt.Printf("Long string: %.5s\n", "HelloWorld")` $\rightarrow$ `Long string: Hello`
	- **控制宽度的修饰符：**
		- 最小字段宽度：在 `%` 与类型说明符之间添加数字 `W`，表示最小宽度。如果输出内容长度不足 `W`，则在**左侧填充空格**，实现右对齐。
		- 左对齐：在宽度数字前添加**负号** `-`，表示输出**左对齐**。
	- **代码示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		  	// 各种格式说明符
		  	fmt.Printf("值 %%v: %v\n", 42)
		  	fmt.Printf("类型 %%T: %T\n", 3.14)
		  	fmt.Printf("十进制整数 %%d: %d\n", 42)
		  	fmt.Printf("浮点数 %%f: %f\n", 3.14159265)
		  	fmt.Printf("字符串 %%s: %s\n", "world")
		  	fmt.Printf("布尔值 %%t: %t\n", true)
		  	var a int
		  	fmt.Printf("指针地址 %%p: %p\n", &a)
		  
		  	fmt.Println()
		  
		  	// 控制精度
		  	fmt.Printf("保留两位小数 %.2f\n", 3.14159)
		  	fmt.Printf("字符串截断 %.5s\n", "HelloWorld")
		  
		  	fmt.Println()
		  
		  	// 控制宽度
		  	fmt.Printf("右对齐 |%6d|\n", 42)
		  	fmt.Printf("左对齐 |%-6d|\n", 42)
		  }
		  
		  ```
- `func Scan(a ...any) (n int, err error)`
  collapsed:: true
	- **作用：**
		- `Scan` 从**标准输入（stdin）**读取文本，并将读取到的**以空格分隔**的值依次存入提供的参数变量中。
		- 换行符会被当作空格处理。
		- 它返回成功读取的项数 `n`。
		- 如果成功读取的数量少于参数个数，`err` 会说明原因（例如输入格式不对或输入提前结束）。
	- **代码示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      var name string
		      var age int
		      fmt.Print("请输入名字和年龄（用空格分隔）：")
		      n, err := fmt.Scan(&name, &age)
		      if err != nil {
		          fmt.Println("输入出错：", err)
		          return
		      }
		      fmt.Printf("成功读取 %d 个值：名字=%s, 年龄=%d\n", n, name, age)
		  }
		  
		  ```
	- **注意：**
		- `fmt.Scan` 会在遇到空格或换行时分割输入。如果你需要读取整行（包括空格），用 `bufio.NewReader(os.Stdin)` 更合适。
		- `fmt.Scan` 只会读取**刚好够参数数量**的数据，后面的数据**不会被丢弃**，而是**留在输入缓冲区中**，下一次再调用 `fmt.Scan` 时，会从那儿继续读取。
		- 输入不足时，程序会等待更多输入。
- `func Fprintf(w io.Writer, format string, a ...any) (n int, err error)`
  collapsed:: true
	- **作用：**`Fprintf` 按照格式说明符格式化内容，并将结果写入 `w`。它返回已写入的字节数，以及可能遇到的写入错误。
- `func Sprintf(format string, a ...any) string`
  collapsed:: true
	- **作用：**`Sprintf` 会根据格式说明符进行格式化处理，并返回生成的字符串。
-