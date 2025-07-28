- **基本概念：**
	- Go 语言通过返回值来显式地传递错误信息，区别于像 Java、Python 等语言中使用异常的方式。
	- 每个可能出错的函数都会返回至少两个值：一个是期望的结果，另一个是 `error` 接口类型的错误。只有当 `error` 为 `nil` 时，才表示函数调用成功，否则需要处理错误。
- **`defer` 关键字：**
	- `defer` 关键字用于推迟函数或方法的执行。
	- 可以将 `defer` 理解为一个“稍后执行”的清单，每写一个 `defer`，就相当于往清单里添加一项任务。当函数即将结束时，Go 会依次执行清单中的任务。
	- 被 `defer` 的函数调用（即延迟调用）会在所在函数执行完毕、准备返回之前执行。
	- 无论函数是否因异常中止，`defer` 的调用都会在函数退出时执行。
	- **最佳实践：**
		- 获取资源对象后，立即使用 `defer` 安排关闭任务。
		- **好处：**
			- **代码紧邻**：将获取资源和释放资源的代码放在一起，简洁明了，易于维护。
			- **防止遗忘**：不再需要担心在函数中某个 `if` 分支里 `return` 后忘记关闭文件。无论 `main` 函数如何退出（正常结束、`return` 或 `panic`），`defer` 语句都会保证执行。
	- **应用场景：**
		- 资源释放（确保文件、数据库连接、网络连接等资源得到关闭）
		- 及时解锁，避免死锁
		- `panic` 后的恢复：结合 `defer` 和 `recover` 可以捕获 `panic`，防止程序崩溃。
	- **后进先出原则：**
		- 如果一个函数中有多个 `defer` 语句，它们会按照“后进先出”的顺序执行：最后声明的 `defer` 最先执行。
	- **参数在 `defer` 声明时立即求值：**
		- 延迟调用的函数参数会在 `defer` 语句声明时就被计算并保存，而不是在函数实际执行时才计算。
- **`panic` 函数：**
	- `panic` 是 Go 的一个内置函数，它会中断程序的正常执行流程。你可以把它想象成一颗“手榴弹”💣，一旦拉响（被调用），它就会立即引爆当前的任务（goroutine），除非被特殊机制“拆除”（`recover`）。
	- 当函数调用 `panic` 时，会依次触发以下事件：
		- **立即终止执行**：当前函数的执行会即刻停止，后续代码不再运行。
		- **执行 `defer` 语句**：
			- 该函数中所有被延迟（`defer`）的调用会按后进先出的顺序立即执行。
			- 这是 `panic` 与 `os.Exit` 的核心区别 —— 后者会直接终止程序，不会执行 `defer`。
		- **沿调用栈向上“冒泡”**：`panic` 状态会传递给调用者，重复执行“中断”和“执行 Defer”的过程，并沿着当前 goroutine 的调用栈逐层向上蔓延。
		- **程序崩溃**：若 “恐慌” 状态蔓延至 goroutine 调用栈顶端仍未被处理，程序就会崩溃，此时会打印 `panic` 传入的值和完整栈追踪信息，并以非零状态码退出。
	- **`panic` 的“冒泡”之旅：**
		- 当一个函数中发生了 `panic`，正常的“函数返回”流程就被打断了，取而代之的是“栈解退”的冒泡过程。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func functionB() {
		      // defer 在 panic "冒泡"离开本函数前执行
		      defer fmt.Println("清理 functionB...")
		      
		      fmt.Println("进入 functionB")
		      panic("发生了一个重大问题！") // panic 被触发！
		      
		      // panic 下面的代码永远不会被执行
		      fmt.Println("离开 functionB") 
		  }
		  
		  func functionA() {
		      defer fmt.Println("清理 functionA...")
		      
		      fmt.Println("进入 functionA")
		      functionB() // 调用 functionB
		      fmt.Println("离开 functionA")
		  }
		  
		  func main() {
		      defer fmt.Println("清理 main...")
		      
		      fmt.Println("进入 main")
		      functionA() // 调用 functionA
		      fmt.Println("离开 main")
		  }
		  
		  // 程序的最终输出会是：
		  进入 main
		  进入 functionA
		  进入 functionB
		  清理 functionB...
		  清理 functionA...
		  清理 main...
		  panic: 发生了一个重大问题！
		  
		  goroutine 1 [running]:
		  main.functionB()
		  	/path/to/your/program.go:12 +0xbf
		  main.functionA()
		  	/path/to/your/program.go:21 +0x7d
		  main.main()
		  	/path/to/your/program.go:28 +0x7d
		  ... (其他信息)
		  ```
		- **第 1 站：`functionB` (恐慌的源头)**
			- 程序执行到 `functionB` 里的 `panic("...")`。
			  logseq.order-list-type:: number
			- `functionB` 的正常执行立即停止。`fmt.Println("离开 functionB")` 这行代码被跳过。
			  logseq.order-list-type:: number
			- 系统开始“解退”`functionB` 的栈帧。在离开之前，它会检查 `functionB` 中有没有 `defer` 语句。
			  logseq.order-list-type:: number
			- 发现 `defer fmt.Println("清理 functionB...")`，于是执行它。屏幕上打印出 `清理 functionB...`。
			  logseq.order-list-type:: number
			- `functionB` 执行完毕，`panic` “冒泡”到它的调用者——`functionA`。
			  logseq.order-list-type:: number
		- **第 2 站：`functionA` (恐慌的传播)**
			- `functionA` 在调用 `functionB()` 的地方接收到了这个 `panic`。
			- `functionA` 的正常执行也立即停止。`fmt.Println("离开 functionA")` 被跳过。
			- 系统开始解退 `functionA` 的栈帧，并检查它的 `defer` 语句。
			- 发现 `defer fmt.Println("清理 functionA...")`，执行它。屏幕上打印出 `清理 functionA...`。
			- `functionA` 执行完毕，`panic` 继续“冒泡”到它的调用者——`main`。
		- **第 3 站：`main` (最后一站)**`
			- main` 函数在调用 `functionA()` 的地方接收到 `panic`。`
			  logseq.order-list-type:: number
			- main` 的执行也停止，`fmt.Println("离开 main")` 被跳过。
			  logseq.order-list-type:: number
			- 系统解退 `main` 的栈帧，检查 `defer`。
			  logseq.order-list-type:: number
			- 发现 `defer fmt.Println("清理 main...")`，执行它。屏幕上打印出 `清理 main...`。
			  logseq.order-list-type:: number
		- **终点：程序崩溃**
			- `main` 是这个 goroutine 的顶层函数，`panic` 已经“冒泡”到顶了，并且没有被“处理”。
			- 程序崩溃。它会打印出 `panic` 的值以及完整的栈追踪信息。
	- **何时应该使用 `panic`？**
		- 在 Go 的设计哲学中，`error` 是处理预期错误的标准方式，而 `panic` 仅用于真正的异常情况。
		- 始终避免使用 `panic` 处理普通错误，这类错误应通过返回 `error` 来处理。`panic` 的合理使用场景仅限于程序遇到不可恢复的严重错误，导致继续运行无意义时，此时应使用它 “快速失败”。典型场景包括关键配置文件加载失败、必要依赖服务不可用等。
- **`recover` 函数：**
	- `panic` 的冒泡行为是可以阻止的。`recover()` 函数可以在 `defer` 中捕获 `panic`，从而“戳破”这个气泡，使程序从恐慌状态恢复，避免程序崩溃并继续正常运行。
	- `recover()` 必须在 `defer` 语句中调用才能生效。如果在非 `defer` 语句中调用 `recover`，它将返回 `nil` 且没有任何作用。
	- 若成功捕获 `panic`，`recover` 会返回传给 `panic` 的值，程序将从触发 `panic` 的函数中提前返回，继续执行该函数调用者的后续代码。若未发生 `panic`，`recover` 返回 `nil`，程序按正常流程继续执行。
- [[errors]]
- [[Go 的自定义错误]]
-