- **作用：**`http.Client` 是 Go 标准库中的一个 HTTP 客户端，用于发送 HTTP 请求。
- **结构体定义与字段解析：**
	- ```go
	  type Client struct {
	  	Transport     RoundTripper
	  	CheckRedirect func(req *Request, via []*Request) error
	  	Jar           CookieJar
	  	Timeout       time.Duration
	  }
	  ```
	- **`Transport RoundTripper`：**这是执行单个 HTTP 事务（请求-响应）的底层机制。`Client` 的大部分高级功能（如连接池、连接保持）都由 `Transport` 实现。如果此字段为 `nil`，则会使用全局默认的 `http.DefaultTransport`。
	- **`CheckRedirect func(req *Request, via []*Request) error`：**一个函数，用于定义客户端如何处理 HTTP 重定向。如果为 `nil`，客户端将采用默认策略，即最多跟随 10 次重定向。你可以通过自定义这个函数来阻止重定向或实现更复杂的逻辑。
	- **`Timeout time.Duration`：**为整个 HTTP 请求设置一个总的超时时间。这个超时覆盖了从建立连接、发送请求、等待响应到读取完响应体的全部过程。如果值为 `0`，则表示没有超时限制。
- **注意：**
	- 使用 `http.Client` 最重要的一个最佳实践：**创建一次，全局复用**。
		- **并发安全**：`http.Client` 是并发安全的，这意味着你可以在多个 goroutine 中同时安全地使用同一个实例。
		- **复用连接池：**`http.Client` 内部的 `Transport` 会维护一个 TCP 连接池，用于缓存和复用已有连接。如果每次请求都新建一个 `Client`，这些连接将无法复用，请求就必须重复执行 TCP 握手和（在 HTTPS 中的）TLS 握手，进而显著增加延迟并降低系统性能。
		- 因此，正确的做法是在应用程序启动时只创建一个 `http.Client` 实例，然后在所有需要发起 HTTP 请求的地方复用它。
	- **超时机制：**`Client` 的 `Timeout` 字段设置的是从开始请求到读取完响应体的**总时间**，它控制从请求开始到响应体完全读取结束的整个过程。如果在读取 `Response.Body` 的过程中超时，读取操作会被中断。如果只需要设置单个阶段（如连接建立）的超时，应通过自定义 `Transport` 来实现。
	-