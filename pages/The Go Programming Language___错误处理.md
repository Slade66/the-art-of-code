- **Go 语言的错误处理方式：**
	- 在 Go 中，错误处理不依赖 `try...catch` 这样的异常捕获机制。
	- Go 语言通过返回值显式传递错误信息，函数返回一个 `error` 类型的值来表示可能发生的错误。
	- 每个可能出错的函数通常会返回至少两个值：一个是期望的结果，另一个是 `error` 类型的错误。
		- `error` 为 `nil`：表示操作成功，没有发生错误。
		  id:: 68f6fc86-a943-4aa2-acf5-90623686ed88
		- `error` 非 `nil`：表示操作失败，出现了错误。
		  id:: 68f6fc8a-603f-49c7-b337-6b1076ea78f7
	- **示例：**
		- ```go
		  // 尝试做某事，它可能成功（返回真实值, nil）
		  // 或者失败（返回零值, error）
		  value, err := doSomething()
		  if err != nil {
		      // 失败了！
		      // 立即处理这个错误（比如：打日志、返回、或panic）
		      return err
		  }
		  // 成功了！
		  // ...继续使用 value...
		  ```
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
	- `panic` 的冒泡行为是可以阻止的。`recover()` 函数可以在 `defer` 中捕获一个正在“冒泡”的 `panic`，从而“戳破”这个气泡，使程序从恐慌状态中恢复，避免崩溃并继续运行。
	- `recover()` 必须在 `defer` 语句中调用才能生效。
		- 为什么？因为当 `panic` 发生时，正常的代码执行流程会立即停止，只有 `defer` 中的任务会继续执行。因此，`defer` 函数是唯一能在 `panic` 发生后、程序崩溃前进行干预的地方。
	- 如果在没有 `panic` 的情况下调用 `recover()`，或在 `defer` 之外调用它，它不会起任何作用，并会返回 `nil`。
	- 如果成功捕获 `panic`，`recover` 会返回传给 `panic` 的值。
	- 处理了下层 `panic` 的函数，自己也无法从发生 `panic` 的那个调用点继续向后执行。
	- **示例：**
		- ```go
		  package main
		  
		  import (
		  	"fmt"
		  	"time"
		  )
		  
		  // 第 3 层：这个函数会触发 panic
		  func panickingFunction() {
		  	fmt.Println(" [3] 进入了会 panic 的函数... ")
		  
		  	// 触发 panic
		  	panic("发生了一个本不该发生的错误")
		  
		  	// --- 关键点 1 ---
		  	// panic 一旦触发，当前函数的执行就已停止。
		  	// 下面这行代码是永远、永远不会被执行的。
		  	fmt.Println(" [3] panic 之后（这行代码永远不会执行）")
		  }
		  
		  // 第 2 层：这个函数负责调用并恢复(recover) panic
		  func recoveringFunction() {
		  	// 使用 defer 来设置一个“安全网”
		  	defer func() {
		  		fmt.Println(" [2] defer 安全网已启动...")
		  
		  		// 调用 recover() 来捕获 panic
		  		if r := recover(); r != nil {
		  			fmt.Printf(" [2] 成功捕获到 panic！值为: %v\n", r)
		  			fmt.Println(" [2] panic 已被处理，程序将从这里恢复并正常返回。")
		  		}
		  	}()
		  
		  	fmt.Println(" [2] 准备调用会 panic 的函数... ")
		  	panickingFunction() // 调用第 3 层函数
		  
		  	// --- 关键点 2 ---
		  	// 当 panickingFunction() 发生 panic 时，执行流会立即跳转到本函数的 defer 代码块。
		  	// 因此，本函数的正常流程也被中断了，下面这行代码同样不会被执行。
		  	fmt.Println(" [2] 调用 panickingFunction() 之后（这行也不会执行）")
		  }
		  
		  // 第 1 层：主函数，调用我们的“恢复函数”
		  func main() {
		  	fmt.Println(" [1] 程序开始... ")
		  
		  	recoveringFunction()
		  
		  	// --- 关键点 3 ---
		  	// 因为 recoveringFunction 内部成功处理了 panic，
		  	// 所以 panic 没有“冒泡”到 main 函数。
		  	// main 函数会认为 recoveringFunction 只是一个正常返回的函数调用。
		  	// 因此，这里的代码会继续执行。
		  	fmt.Println(" [1] recoveringFunction 已返回，程序继续执行...")
		  	time.Sleep(1 * time.Second)
		  	fmt.Println(" [1] 程序正常结束。")
		  }
		  ```
- [[Go 的自定义错误]]
-