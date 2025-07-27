`defer` 关键字
heading:: true
	- `defer` 用于延迟执行一个函数调用，无论当前函数是否因异常中止，该调用都会在函数退出时被执行。通常用于资源释放，以确保关键操作不会被遗漏。
- `panic` 函数
  heading:: true
	- `panic` 是 Go 的内置函数，用于触发运行时错误（如数组越界、空指针引用），也可由开发者主动调用。
	- 当一个函数触发 `panic` 时，它会立即中断当前函数的执行，`panic` 之后的代码将不会再被执行。随后，`panic` 会沿调用栈逆序向上“冒泡”，在此过程中，当前 `goroutine` 中已注册的 `defer` 函数会按后进先出的顺序依次执行。如果这个 `panic` 在回溯过程中没有被任何 `recover` 捕获，它最终将导致整个 `goroutine` 崩溃。
	- 通常不建议滥用 `panic`。只有在程序遇到不可恢复的严重错误、继续运行已无意义的情况下，才应使用，例如关键配置文件加载失败或必要服务不可用等。
	- **语法格式：**`panic(interface{})`
- `recover` 函数
  heading:: true
	- `recover` 也是 Go 的内置函数，用于在 `defer` 函数中捕获 `panic`，从而防止程序崩溃。
	- 只有在 `defer` 修饰的函数中调用 `recover` 才有效。如果在非 `defer` 函数中调用 `recover`，它将返回 `nil` 且没有任何作用。
	- 若成功捕获 `panic`，`recover` 会返回传入的值，程序将从触发 `panic` 的函数中提前返回，继续执行该函数调用者的后续代码。若未发生 `panic`，`recover` 返回 `nil`，程序按正常流程继续执行。
	- **示例代码：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      fmt.Println("开始执行 main 函数")
		      safeExecute()
		      fmt.Println("main 函数执行完毕")
		  }
		  
		  func safeExecute() {
		      defer func() {
		          if r := recover(); r != nil {
		              fmt.Println("捕获到了一个 panic:", r)
		          }
		      }()
		  
		      fmt.Println("safeExecute: 准备触发一个 panic")
		      panic("这是一个测试 panic")
		      fmt.Println("safeExecute: panic 之后") // 不会被执行
		  }
		  ```
-