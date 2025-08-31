- **作用：**向指定的 URL 发送一个 POST 请求，并返回服务器的响应。
- **方法签名：**
	- ```go
	  func Post(url, contentType string, body io.Reader) (resp *Response, err error)
	  ```
- **参数：**
	- **`url string`：**要发送 POST 请求的目标 URL。
	- **`contentType string`：**请求主体的数据类型，例如 `application/json`。
	- **`body io.Reader`：**请求主体数据的读取器。
- **返回值：**
	- **`resp *Response`：**[[type Response]]
	- **`err error`：**如果请求过程中发生错误（如网络问题、无效的 URL 等），该返回值将非空，否则为 `nil`。
- **核心执行流程：**
	- 直接调用 `DefaultClient.Post(url, contentType, body)`。
	  logseq.order-list-type:: number
	- 将 `DefaultClient.Post` 的返回值原封不动地返回。
	  logseq.order-list-type:: number
- **注意：**
	- **自动关闭：**如果传递给 `Post` 函数的请求体（`body` 参数）实现了 `io.Closer` 接口，那么在请求完成后，函数会自动调用其 `Close` 方法。
	- **自定义配置：**`Post` 函数默认使用 `http.DefaultClient`，功能较为基础。如果需要自定义请求头、设置超时或传递上下文（`context`），建议使用 `http.NewRequest` 或 `http.NewRequestWithContext` 创建请求，再配合 `DefaultClient.Do` 执行。