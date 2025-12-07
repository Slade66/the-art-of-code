goroutine
heading:: true
	- goroutine 是 Go 语言中的一种轻量级的线程，也称为协程，可以让程序同时处理多个任务。
	- 其内存开销远小于传统线程，因此可以同时运行成几十万甚至上百万个 goroutine，而不会像传统线程那样消耗大量系统资源。
	- goroutine 由 Go 运行时调度和管理：它的操作都在用户态下完成，避免了像传统线程那样切换到内核态，显著减少了上下文切换的开销，更加高效。
	- 启动很简单：只需在函数调用前加上 `go` 关键字，就能创建一个新的 goroutine。
	- `go` 语句不能有返回值，因为其启动的 goroutine 是异步执行的，调用方无法立即获取其结果。
	- `main` 函数在一个 goroutine 中执行，而 `go` 语句会创建额外的 goroutine。
	- **用匿名函数启动 goroutine：**
		- ```go
		  go func(s string) {
		    fmt.Printf("%s from goroutine\n", s)
		  }("haha")
		  ```
- channel
  heading:: true
	- `channel` 最主要的作用是让不同的 goroutine 之间能够安全地传递数据。一个 goroutine 可以将数据“发送”到通道中，另一个 goroutine 可以从通道中“接收”数据。
	- 除了数据传输，`channel` 也天然地提供了同步功能。发送或接收操作会阻塞，直到另一个 goroutine 准备好进行相应的操作，从而实现 goroutine 间的协调。
	- **创建 channel：**
		- ```go
		  ch := make(chan ElementType, [capacity])
		  
		  var ch chan ElementType
		  ch = make(chan ElementType, [capacity])
		  ```
		- `ElementType`：通道中传输的数据类型。通道具有类型限制，只能传递该类型的值。
		- `[capacity]`：可选参数，表示通道的缓冲区大小。
			- 如果省略 `capacity`（或设置为 `0`），则创建的是无缓冲通道（unbuffered channel）。此时发送和接收操作必须同步进行：发送方在发送数据时，必须有接收方准备接收，否则发送会阻塞；同样，接收方在接收数据时，必须有发送方准备发送，否则接收会阻塞。
			- 如果指定了大于 `0` 的 `capacity`，则创建的是有缓冲通道（buffered channel）。它可以存储指定数量的元素：当缓冲区未满时，发送操作不会阻塞；当缓冲区不为空时，接收操作也不会阻塞。
	- **数据的发送和接收：**
		- **发送数据：**`ch <- 42 // 把 42 发送到 ch`
		- **接收数据：**`value := <-ch // 从通道 ch 接收数据并赋值给变量 value`
		- **限定通道只能用于发送或接收：**
			- **只发送通道：**
				- ```go
				  func sendData(ch chan<- int) { // ch 是一个只发送 int 的通道
				      ch <- 10
				  }
				  ```
			- **只接收通道：**
				- ```go
				  func receiveData(ch <-chan string) { // ch 是一个只接收 string 的通道
				      msg := <-ch
				      fmt.Println(msg)
				  }
				  ```
			- 箭头（`<-`）写在通道右边是发送，写在通道左边是接收。
	- **关闭通道：**
		- 使用内置的 `close` 函数关闭通道。
		- 语法：`close(ch)`
		- **注意：**
			- 只能由发送方关闭通道，接收方尝试关闭通道会导致运行时 panic。
			- 重复关闭同一个通道会导致运行时 panic。
			- 关闭通道后，仍然可以从通道接收剩余的数据。当所有数据都被接收后，继续接收将立即返回该类型零值，并且第二个返回值 `ok` 会是 `false`。
- sync 包
  heading:: true
	- `type Once struct {...}`
		- `func (o *Once) Do(f func())`
			- `Do` 方法接受一个没有参数也没有返回值的函数 `f` 作为它的参数。
			- 当你调用 `once.Do(f)` 时，`sync.Once` 会检查它内部的一个状态。
				- **第一次调用时：**它会锁定，然后执行你传入的函数 `f`，执行完毕后，它会标记为“已完成”并解锁。
				- **后续所有调用时：**无论是哪个 goroutine 发起的调用，`Do` 方法都会看到“已完成”的标记，然后直接返回，什么也不做。
-