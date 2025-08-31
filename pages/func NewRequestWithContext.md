- **作用：**创建一个新的 `*http.Request` 实例，该实例与一个 `context.Context` 关联，用于控制请求的整个生命周期。
- **方法签名：**
	- ```go
	  func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error)
	  ```
- **参数：**
	- **`ctx context.Context`**：用于控制请求生命周期的上下文。在获取连接、发送请求、读取响应等过程中，它可以传递取消信号。如果传入 `nil`，会直接导致程序 `panic`。
	- **`method string`**：HTTP 请求方法，如 `"GET"`、`"POST"`、`"PUT"` 等。若传入空字符串，则默认使用 `"GET"`。
	- **`url string`**：请求的目标 URL。函数会对其进行解析，若解析失败则返回错误。
	- **`body io.Reader`**：请求体，可选参数。传入 `nil` 表示没有请求体。
- **返回值：**
	- **`*Request`：**一个指向新创建的 `http.Request` 实例的指针。
	- **`error`：**如果在创建请求过程中发生错误，则返回一个非 `nil` 的 `error`。
- **代码示例：**
	- 下面是一个使用 `context.WithTimeout` 创建一个在 2 秒后自动取消的 HTTP GET 请求的示例。我们请求一个需要 5 秒才能响应的服务器，因此请求会因为超时而失败。
	- ```go
	  package main
	  
	  import (
	  	"context"
	  	"fmt"
	  	"io"
	  	"net/http"
	  	"time"
	  )
	  
	  // slowHandler 模拟一个耗时 5 秒的服务器端点
	  func slowHandler(w http.ResponseWriter, r *http.Request) {
	  	fmt.Println("服务器端：开始处理请求...")
	  	time.Sleep(5 * time.Second)
	  	w.Write([]byte("处理完成！"))
	  	fmt.Println("服务器端：请求处理完毕。")
	  }
	  
	  func main() {
	  	// 启动一个模拟慢速响应的服务器
	  	go http.ListenAndServe(":8080", http.HandlerFunc(slowHandler))
	  	time.Sleep(100 * time.Millisecond) // 确保服务器已启动
	  
	  	// 创建一个 2 秒后会自动过期的 context
	  	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	  	defer cancel() // 确保在 main 函数退出时释放 context 相关资源
	  
	  	// 使用 NewRequestWithContext 创建一个带超时控制的请求
	  	req, err := http.NewRequestWithContext(ctx, "GET", "http://localhost:8080", nil)
	  	if err != nil {
	  		fmt.Printf("创建请求失败: %v\n", err)
	  		return
	  	}
	  
	  	// 使用默认的 http.Client 发送请求
	  	fmt.Println("客户端：发送请求...")
	  	resp, err := http.DefaultClient.Do(req)
	  
	  	// 检查返回的错误
	  	if err != nil {
	  		// 由于 context 超时，这里会捕获到错误
	  		fmt.Printf("客户端：请求失败: %v\n", err) // 输出：... context deadline exceeded
	  		return
	  	}
	  	defer resp.Body.Close()
	  
	  	// 如果没有错误，则读取响应（在此示例中，这部分代码不会执行）
	  	body, _ := io.ReadAll(resp.Body)
	  	fmt.Printf("客户端：收到响应: %s\n", body)
	  }
	  ```
- **注意：**
	- **Context 的作用域**：`context` 的生命周期控制了整个请求过程，从 DNS 解析、TCP 连接、发送请求数据到接收响应体。一旦 `context` 被取消，整个流程都会被中断。
	- **与 `http.NewRequest` 的区别**：`http.NewRequest` 创建的请求默认关联 `context.Background()`，意味着它没有超时或取消机制。`NewRequestWithContext` 是专门为了创建可控生命周期的请求而设计的。