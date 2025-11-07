- **作用：**
	- `http.Response` 结构体用于封装从 HTTP 服务器接收到的完整响应。当 Go 的 `http.Client` 发送请求后，其返回值就是 `*http.Response` 类型。
	- 该结构体封装了所有来自服务器的响应信息，主要包括：
		- **状态行：**包含状态码（如 `200 OK`），表示请求处理的结果。
		- **响应头：**键值对形式的元数据，如 `Content-Type`、`Content-Length` 和 `Set-Cookie`，提供了关于响应内容的额外信息。
		- **响应体：**服务器返回的实际数据，例如 HTML、JSON 或文件流。响应体是一个可读的流，这意味着你可以在接收到响应头后立即开始处理数据，无需等待整个响应体下载完成，这对于处理大文件或数据流非常高效。
- **结构体定义与字段解析：**
	- ```go
	  type Response struct {
	  	// 作用：包含完整的 HTTP 状态文本，由状态码和原因短语组成。
	  	// 示例："200 OK", "404 Not Found", "502 Bad Gateway"。
	  	// 注意：这是一个供人类阅读的字符串。在程序中进行逻辑判断时，应使用 StatusCode 字段。
	  	Status string
	  
	  	// 作用：提供数字形式的 HTTP 响应状态码。
	  	// 示例：200, 404, 502。
	  	// 注意：这是程序中判断请求成功、失败或进行其他逻辑分支的权威依据，例如 `if resp.StatusCode >= 200 && resp.StatusCode < 300`。
	  	StatusCode int
	  
	  	// 作用：表示响应所使用的协议版本字符串。
	  	// 示例："HTTP/1.1", "HTTP/2.0"。
	  	// 注意：通常与 ProtoMajor 和 ProtoMinor 字段一起提供协议版本信息。
	  	Proto string
	  
	  	// 作用：表示协议的主版本号。
	  	// 示例：对于 "HTTP/1.1"，值为 1。
	  	// 注意：用于需要精确版本判断的场景，比直接解析 Proto 字符串更高效可靠。
	  	ProtoMajor int
	  
	  	// 作用：表示协议的次版本号。
	  	// 示例：对于 "HTTP/1.1"，值为 1。
	  	// 注意：同上，与 ProtoMajor 配合使用。
	  	ProtoMinor int
	  
	  	// 作用：以键值对的形式存储所有 HTTP 响应头。
	  	// 示例：`contentType := resp.Header.Get("Content-Type")`。
	  	// 注意：Header 的键是经过标准化的（首字母大写格式，如 "Content-Type"）。当 Header 中的信息与结构体中的特定字段（如 ContentLength）冲突时，后者的优先级更高。
	  	Header Header
	  
	  	// 作用：提供对响应体的流式读取访问。它是一个 io.ReadCloser 接口。
	  	// 示例：`jsonData, err := io.ReadAll(resp.Body)`。
	  	// 注意：【极其重要】必须在使用完毕后手动调用 `Close()` 方法，强烈推荐使用 `defer resp.Body.Close()` 来防止连接和内存泄漏。即使响应没有内容，Body 字段也永远不会是 nil。
	  	Body io.ReadCloser
	  
	  	// 作用：记录响应体的确切长度（以字节为单位）。
	  	// 示例：如果服务器返回一个 1024 字节的图片，此值为 1024。
	  	// 注意：如果值为 -1，表示响应体长度未知，这通常发生在服务器使用 `Transfer-Encoding: chunked` 时。
	  	ContentLength int64
	  
	  	// 作用：记录应用于消息体的传输编码，按从外到内的顺序排列。
	  	// 示例：当服务器使用分块传输时，值为 `[]string{"chunked"}`。
	  	// 注意：值为 nil 等同于 `[]string{"identity"}`，表示内容没有经过特殊的传输编码。
	  	TransferEncoding []string
	  
	  	// 作用：一个布尔标志，指示服务器是否希望在响应发送完毕后关闭连接。
	  	// 示例：如果响应头中包含 `Connection: close`，此值为 true。
	  	// 注意：这只是服务器的一个建议。连接的最终管理由客户端的 http.Transport 决定，它可能会为了复用而忽略此建议。
	  	Close bool
	  
	  	// 作用：一个布尔标志，报告响应是否由 http 包自动解压。
	  	// 示例：服务器返回了 gzip 压缩的数据，客户端自动解压后，此值为 true。
	  	// 注意：当此值为 true 时，你从 Body 读取的是解压后的数据，且 `ContentLength` 会被设为 -1，原始的 `Content-Encoding` 头也会被移除。
	  	Uncompressed bool
	  
	  	// 作用：存储在响应体之后 发送的 "拖挂" 头信息。
	  	// 示例：在流式传输结束时，服务器可能会通过拖挂头发送一个数据校验和，如 `X-Checksum: ...`。
	  	// 注意：必须在 `resp.Body` 被完全读取（即 `Read` 方法返回 `io.EOF` 错误）之后，才能从此字段获取到有效值。
	  	Trailer Header
	  
	  	// 作用：一个反向指针，指向生成此响应的原始 `http.Request` 对象。
	  	// 示例：`log.Printf("Request to %s got status %d", resp.Request.URL, resp.StatusCode)`。
	  	// 注意：此字段仅在客户端请求中才会被填充。并且其 `Body` 字段为 nil，因为请求体已经被发送出去了。
	  	Request *Request
	  
	  	// 作用：包含关于接收此响应的 TLS（HTTPS）连接的详细信息。
	  	// 示例：`if resp.TLS != nil { fmt.Println(resp.TLS.Version) }` 可以获取 TLS 版本号。
	  	// 注意：如果连接不是 HTTPS，此字段为 nil。
	  	TLS *tls.ConnectionState
	  }
	  ```
- **注意：**
	- 不要在循环中使用 `defer` 来关闭 `res.Body`，因为这样会导致关闭操作延迟到函数结束时才执行。对于长时间运行的程序，这会造成连接或文件描述符被耗尽。正确的做法是在处理完响应后立即调用 `res.Body.Close()`。
-