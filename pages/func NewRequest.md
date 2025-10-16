- **作用：**创建一个新的 `http.Request` 实例。
- **方法签名：**
	- ```go
	  func NewRequest(method, url string, body io.Reader) (*Request, error)
	  ```
- **参数：**
	- `method string`：指定 HTTP 请求的方法（例如 "GET", "POST", "PUT", "DELETE" 等）。方法名是大小写敏感的，通常使用大写形式。
	- `url string`：指定请求的目标 URL。这个 URL 必须是能够被解析的有效 URL 字符串。
	- `body io.Reader`：一个 `io.Reader` 接口，用于提供请求的主体（request body）。对于没有请求体的 HTTP 方法（如 "GET"），这个参数可以传入 `nil`。
- **返回值：**
	- `*Request`：如果成功，返回一个指向新创建的 `http.Request` 结构体的指针。你可以后续修改这个 `Request` 对象，例如添加请求头（Headers）。
	- `error`：如果在创建请求过程中发生错误（例如，无效的 HTTP 方法或无法解析的 URL），则返回一个非 `nil` 的 `error`。
- **代码示例：**
	- 下面是一个完整的示例，展示了如何创建一个 POST 请求，并为其添加自定义的请求头，然后使用 `http.DefaultClient` 发送它。
	- ```go
	  package main
	  
	  import (
	  	"fmt"
	  	"io"
	  	"net/http"
	  	"strings"
	  )
	  
	  func main() {
	  	// 准备请求体。strings.NewReader 实现了 io.Reader 接口。
	  	requestBody := strings.NewReader(`{"name": "Gemini", "project": "Go Note Generator"}`)
	  
	  	// 1. 使用 http.NewRequest 创建一个 POST 请求
	  	// 参数：方法 ("POST"), URL, 请求体
	  	req, err := http.NewRequest("POST", "https://httpbin.org/post", requestBody)
	  	if err != nil {
	  		fmt.Printf("创建请求失败: %s\n", err)
	  		return
	  	}
	  
	  	// 2. (可选) 为请求设置自定义的 Header
	  	req.Header.Set("Content-Type", "application/json")
	  	req.Header.Set("User-Agent", "Go-Note-Generator/1.0")
	  	req.Header.Set("X-Custom-Header", "my-custom-value")
	  
	  	// 3. 使用一个 HTTP client 来发送这个请求
	  	// http.DefaultClient 是一个默认可用的客户端
	  	client := http.DefaultClient
	  	resp, err := client.Do(req)
	  	if err != nil {
	  		fmt.Printf("发送请求失败: %s\n", err)
	  		return
	  	}
	  	defer resp.Body.Close() // 确保在函数结束时关闭响应体
	  
	  	// 4. 读取并打印响应体内容
	  	fmt.Println("响应状态码:", resp.Status)
	  	responseBody, err := io.ReadAll(resp.Body)
	  	if err != nil {
	  		fmt.Printf("读取响应体失败: %s\n", err)
	  		return
	  	}
	  	fmt.Println("响应内容:")
	  	fmt.Println(string(responseBody))
	  }
	  ```
- **注意：**
	- **不会立即发送**：`http.NewRequest` 只负责创建请求对象，并不会执行任何网络操作。实际的请求发送是由 `http.Client` 的 `Do` 方法完成的。这给了你一个在发送前修改请求（如添加 Header、Cookie 等）的机会。
	- **请求体（Body）**：传入的 `body` 必须是 `io.Reader` 类型。常见的使用方式包括 `strings.NewReader`（用于字符串）、`bytes.NewBuffer`（用于字节切片）或打开的文件句柄（`*os.File`）。
	- **Context 管理**：自 Go 1.7 起，`Request` 结构体包含了上下文（`context.Context`）。如果你需要管理请求的超时、取消等，应该使用 `http.NewRequestWithContext` 来创建请求，这样可以传入一个自定义的 `context`。直接使用 `http.NewRequest` 创建的请求会默认使用 `context.Background()`。
	-