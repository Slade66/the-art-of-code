- **作用：**
	- `context.Done()` 返回一个只读的通道。当任务被取消时，该通道会被关闭。通过关闭通道这一动作，可以向所有监听它的 Goroutine 广播一个信号：任务结束了，请尽快清理并退出。
- **原理：**
	- `Done()` 返回的类型是 `<-chan struct{}`。
	- **当任务正在进行时：**这个 Channel 是打开的。如果你试图从里面读数据（`<-ctx.Done()`），代码会阻塞。
	- **当任务需要停止时：**
		- 当调用了 `cancel()` 或超时，这个 Channel 会被关闭。
		- 在 Go 中，从一个已关闭的 Channel 读取数据，不会阻塞，而是会立即返回（返回零值）。
		- 所有卡在 `<-ctx.Done()` 的 Goroutine 会同时“苏醒”，继续执行后面的清理代码。
- **关闭通道的情况：**
	- WithCancel 会在调用 cancel 时关闭 Done 通道；
	- WithDeadline 会在截止时间到期时关闭 Done 通道；
	- WithTimeout 会在超时时间到期时关闭 Done 通道。
- **用法：**在执行工作时，通过 select 监听 Done() 通道的信号，一旦收到取消信号，尽快停止操作并返回。
- **代码示例：**
	- ```go
	  package main
	  
	  import (
	  	"context"
	  	"fmt"
	  	"time"
	  )
	  
	  // worker 是一个典型的受控协程
	  func worker(ctx context.Context, name string) {
	  	for {
	  		select {
	  		// 【核心代码】：监听停止信号
	  		// 每次循环前，都会先看一眼这里有没有信号
	  		case <-ctx.Done():
	  			fmt.Printf("⛔ %s: 收到老板信号 (%v)，停工清理！\n", name, ctx.Err())
	  			return // 必须 return 结束协程，防止内存泄漏
	  
	  		// 【业务逻辑】：如果没收到停止信号，就干活
	  		default:
	  			fmt.Printf("🔨 %s: 正在搬砖...\n", name)
	  			time.Sleep(500 * time.Millisecond) // 模拟耗时工作
	  		}
	  	}
	  }
	  
	  func main() {
	  	// 设定一个 2 秒后自动超时的 Context
	  	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	  
	  	// 最佳实践：虽然 Timeout 会自动关闭，但显式 defer cancel 依然是好习惯（用于释放计时器资源）
	  	defer cancel()
	  
	  	fmt.Println("--- 开始工作 ---")
	  
	  	// 启动协程，把 ctx 传进去
	  	go worker(ctx, "老王")
	  
	  	// 主程等待 3 秒，观察工人是否会在 2 秒时准时停下
	  	time.Sleep(3 * time.Second)
	  	fmt.Println("--- 主程结束 ---")
	  }
	  
	  ```
-