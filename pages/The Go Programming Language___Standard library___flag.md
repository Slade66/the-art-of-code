- **作用：**
	- 这个
	- 很多程序，尤其是服务端程序或者命令行工具，都需要在启动时接收外部传入的参数，比如配置文件路径、服务端口号、是否开启调试模式等。
	- `flag` 包是 Go 标准库自带的，专门用来解析命令行参数（command-line flag parsing）。
- **为什么要叫 flag？**
	- 在这个 Go `flag` 包的上下文中，“flag” 是一个专业术语，它指的是在命令行中用来控制程序行为的特定参数。
- **什么是命令行标志？**
	- ```bash
	  ./my-app -port=8080 -debug=true data.file
	  ```
	- `-port=8080` 和 `-debug=true` 就是标志（flags）。
	- `data.file` 就是非标志参数。
	- `flag` 包能帮你自动解析出 `port` 是 `8080`，`debug` 是 `true`。
- **使用步骤：**
	- **定义：**在程序里定义你期望接收哪些标志，它们的名称、默认值和帮助信息。
	- **解析：**调用 `flag.Parse()`，这个函数会去读取真正的命令行参数 (来自 `os.Args[1:]`)，并把值赋给你定义的变量。
	- **使用：**在 `flag.Parse()` 之后，你就可以像使用普通变量一样使用这些标志的值了。
- **定义标志的方式：**
	- **方式一：函数返回指针**
		- 像 `flag.String()`、`flag.Int()`、`flag.Bool()` 这类函数。它们会返回一个指向该值的指针。
		- ```go
		  // 1. 定义：nFlag 是一个 *int 类型的指针
		  var nFlag = flag.Int("n", 1234, "flag n 的帮助信息")
		  var sFlag = flag.String("s", "default", "flag s 的帮助信息")
		  
		  func main() {
		      // 2. 解析
		      flag.Parse()
		  
		      // 3. 使用：注意要用 * 来解引用获取指针指向的值
		      fmt.Println("n has value:", *nFlag)
		      fmt.Println("s has value:", *sFlag)
		  }
		  ```
	- **方式二：绑定到现有变量**
		- 有时候你可能更希望把值直接绑定到一个已经声明好的变量上，而不是使用指针。这时你可以用 `flag.IntVar()`、`flag.StringVar()` 这类 `Var` 结尾的函数。
		- ```go
		  // 1. 定义：先声明变量
		  var flagvar int
		  var sFlag string
		  
		  func init() {
		      // 1. 定义：将标志 "flagname" 绑定到 &flagvar
		      flag.IntVar(&flagvar, "flagname", 1234, "flagname 的帮助信息")
		      flag.StringVar(&sFlag, "s", "default", "flag s 的帮助信息")
		  }
		  
		  func main() {
		      // 2. 解析
		      flag.Parse()
		  
		      // 3. 使用：直接使用变量名，不需要指针
		      fmt.Println("flagvar has value:", flagvar)
		      fmt.Println("sFlag has value:", sFlag)
		  }
		  ```
	- **建议：**`Var` 方式（方式二）通常更受青睐，因为它避免了在代码里到处使用指针 (`*`)，让代码更整洁。
- **命令行语法：**
	- `flag` 包支持多种命令行语法：
		- `-flag` —— 开关式：相当于把开关拨到 on（对布尔类型相当于 true）。
		- `--flag` —— 等同于上面（两个破折号只是外观不同）。
		- `-flag=x` —— 把值设置为 x。
		- `-flag x` —— 空格分隔的写法。
	- **示例：**
		- ```go
		  ./app -port=8080 -debug
		  ./app --name=小泽 -timeout 5s
		  ```
	- **布尔标志的陷阱：**
		- 对于非布尔标志，`-flag=x` 和 `-flag x` 都可以。
		- 但对于布尔标志，你必须使用 `-flag=value` 的形式来设置它。
	- **整数和布尔参数的可接受形式：**
		- **整数：**
			- 可写成十进制：`1234`
			- 八进制（前导 0）：`0664`（注意：以 0 开头表示八进制）
			- 十六进制（`0x` 前缀）：`0x1234`
			- 可以为负数：`-42`
		- **布尔值：**
			- 1, 0
			- t, f, T, F
			- true, false（各种大小写形式都可以：TRUE, False, 等）
	- **Duration 类型的参数：**
		- Duration 类型的 flag 使用 `time.ParseDuration` 的规则：像 `300ms`、`2s`、`1m30s`、`2h45m` 都合法。
-