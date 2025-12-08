- **作用：**用于获取 Context 的结束原因。返回一个解释为什么 `Done()` 通道被关闭了的错误。
- **返回值：**
	- `error`：
		- `nil`：Context 还没有结束。此时 `Done()` 通道还是打开的（阻塞状态）。
		- `context.Canceled`：Context 是被显式取消的。
			- 通常是因为调用了 `cancel()` 函数（由 `context.WithCancel` 返回）。
			- 通常是上层不再需要结果了，安全退出。
		- `context.DeadlineExceeded`：Context 是因为超时结束的。
			- 通常是因为到了 `WithDeadline` 设定的时间点，或者超过了 `WithTimeout` 设定的时长。
			- 操作太慢了，可能需要记录日志或监控告警。
- **注意：**
	- 如果 Context 还没结束，调用 `ctx.Err()` 不会阻塞，它会立即返回 `nil`。
- **代码示例：**
	- 最常见的用法是在 `select` 监听 `Done()` 信号后，立即调用 `Err()` 来判断具体的错误类型，以便做不同的处理（例如：如果是超时可能需要重试，如果是取消则直接退出）。
	- ```go
	  package main
	  
	  import (
	  	"context"
	  	"fmt"
	  	"time"
	  )
	  
	  func doWork(ctx context.Context) {
	  	select {
	  	case <-time.After(2 * time.Second):
	  		fmt.Println("工作完成")
	  	case <-ctx.Done():
	  		// 任务被打断了，我们需要知道原因
	  		err := ctx.Err()
	  		
	  		switch err {
	  		case context.Canceled:
	  			fmt.Println("工作停止：被用户取消了")
	  		case context.DeadlineExceeded:
	  			fmt.Println("工作停止：超时了")
	  		}
	  	}
	  }
	  
	  func main() {
	  	// 场景 1：模拟超时
	  	ctx1, cancel1 := context.WithTimeout(context.Background(), 1*time.Second)
	  	defer cancel1()
	  	
	  	fmt.Println("--- 场景 1 ---")
	  	doWork(ctx1)
	  
	  	// 场景 2：模拟手动取消
	  	ctx2, cancel2 := context.WithCancel(context.Background())
	  	
	  	fmt.Println("\n--- 场景 2 ---")
	  	go func() {
	  		time.Sleep(500 * time.Millisecond)
	  		cancel2() // 手动取消
	  	}()
	  	doWork(ctx2)
	  }
	  ```
-