- **作用：**
	- `Do` 方法的核心作用是发送一个 `*http.Request` 请求，并根据客户端 `http.Client` 配置的策略（如重定向、Cookie 管理、认证等）执行该请求，最终返回一个 `*http.Response` 响应。
- **方法签名：**
	- ```go
	  func (c *Client) Do(req *Request) (*Response, error)
	  ```
- **参数：**
	- `req *Request`：一个指向 `http.Request` 结构体的指针。[[type Request]]
- **返回值：**
	- `*Response`：一个指向 `http.Response` 结构体的指针。如果错误为 `nil`，则该返回值必然非 `nil`，并包含一个需要被调用者关闭的响应体 `Body`。
	  id:: 7145568c-7bf6-4b2b-89b1-c2242557e33b
	- `error`：在请求过程中发生客户端策略错误（如 `CheckRedirect` 拒绝重定向）或底层网络错误（如无法建立连接、DNS 解析失败、超时）时，会返回一个非 `nil` 的 `error`。
- **代码示例：**
	- 下面是一个完整且可运行的示例，演示了如何使用 Do 方法发送一个 GET 请求并处理响应。
	- ```go
	  package main
	  
	  import (
	  	"fmt"
	  	"io"
	  	"log"
	  	"net/http"
	  	"time"
	  )
	  
	  func main() {
	  	// 1. 创建一个 http.Client 实例
	  	// Client 是并发安全的，可以复用
	  	client := &http.Client{
	  		Timeout: 10 * time.Second, // 设置请求超时
	  	}
	  
	  	// 2. 创建一个 http.Request 实例
	  	// 这里我们请求一个公共的 JSON API
	  	req, err := http.NewRequest("GET", "https://httpbin.org/get", nil)
	  	if err != nil {
	  		log.Fatalf("创建请求失败: %v", err)
	  	}
	  
	  	// 可以自定义请求头
	  	req.Header.Add("Accept", "application/json")
	  	req.Header.Add("User-Agent", "Go-Client/1.0")
	  
	  	// 3. 使用 client.Do 方法发送请求
	  	fmt.Println("正在发送请求...")
	  	resp, err := client.Do(req)
	  	if err != nil {
	  		log.Fatalf("发送请求失败: %v", err)
	  	}
	  
	  	// 4. 函数退出前，必须关闭 resp.Body
	  	// 这是使用 Do 方法时最重要的一个点！
	  	defer resp.Body.Close()
	  
	  	// 5. 检查响应状态码
	  	fmt.Printf("收到响应状态码: %d %s\n", resp.StatusCode, resp.Status)
	  
	  	// 6. 读取响应体
	  	body, err := io.ReadAll(resp.Body)
	  	if err != nil {
	  		log.Fatalf("读取响应体失败: %v", err)
	  	}
	  
	  	// 7. 打印响应内容
	  	fmt.Println("响应内容:")
	  	fmt.Println(string(body))
	  }
	  ```
- **注意：**
	- **必须关闭 `Response.Body`**：当 `error` 为 `nil` 时，返回的 `*Response` 会包含一个非空的 `Body`（`io.ReadCloser`）。若不在使用完后调用 `resp.Body.Close()`，TCP 连接将无法回收复用，可能导致文件描述符和内存耗尽，从而引发资源泄漏。推荐的做法是使用 `defer resp.Body.Close()`。
	- **错误处理**：`Do` 方法返回的 `error` 通常表示网络层面的问题（如 DNS 查询失败、连接被拒、超时）或客户端配置错误。服务器返回的 4xx 或 5xx 状态码不被视为错误，此时 `err` 依然是 `nil`。你需要通过检查 `resp.StatusCode` 来判断请求是否在业务上成功。
	- **超时**：建议为 `http.Client` 设置一个 `Timeout`。这个超时涵盖了从连接、请求到读取响应头的整个过程。如果没有设置，请求可能会永远阻塞。
	- **请求体 `Request.Body`**：`Do` 方法的底层 `Transport` 会负责关闭你提供的 `req.Body`，无论请求成功与否。你不需要在调用 `Do` 之后手动关闭 `req.Body`。
-