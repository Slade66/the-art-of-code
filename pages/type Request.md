- **作用：**`Request` 结构体封装了从客户端发来的 HTTP 请求的全部信息。
- **结构体定义与字段解析：**
	- ```go
	  type Request struct {
	  	// 作用：指定 HTTP 请求的方法，例如 "GET", "POST", "PUT" 等，决定了请求的类型。
	  	// 注意：
	  	// - 对于客户端，如果此字段为空，默认使用 "GET" 方法。
	  	// - 服务器端收到的请求，此字段会精确地反映客户端所用的方法。
	  	Method string
	  
	  	// 作用：代表请求的 URL，是整个请求的核心部分，包含了请求的路径、查询参数等。
	  	// 注意：
	  	// - 在服务器端，URL 的 Host 部分是空的，主机信息会被单独提取到 Request.Host 字段。
	  	// - 在客户端，URL 的 Host 字段则用于指定连接的服务器。
	  	URL *url.URL
	  
	  	// 作用：描述请求所使用的 HTTP 协议版本（如 "HTTP/1.1", "HTTP/2.0"）。
	  	// 注意：
	  	// - 此字段由服务器在接收请求时自动填充。
	  	// - 对于客户端请求，这些字段会被 Go 自动忽略，客户端会根据需要自动选择合适的协议版本。
	  	Proto      string
	  	ProtoMajor int
	  	ProtoMinor int
	  
	  	// 作用：存储请求的 HTTP 头部，包含各种元数据，如 Content-Type、Authorization 等。
	  	// 注意：
	  	// - HTTP 头部名称不区分大小写，Go 库会自动进行规范化。
	  	// - 服务器端收到的请求，"Host" 头部会被单独提取到 Request.Host 字段。
	  	Header Header
	  
	  	// 作用：承载 POST、PUT 等请求的请求主体数据，比如表单数据或 JSON。
	  	// 注意：
	  	// - Body 实现了 io.ReadCloser 接口，只能被读取一次。
	  	// - 服务器端，即使没有请求体，Body 也非 nil，但会立即返回 EOF。
	  	// - 客户端则用 nil 表示无请求体。
	  	Body io.ReadCloser
	  
	  	// 作用：一个可选的函数，当客户端需要因为重定向等原因重新发送请求时，用于生成一个新的 Body。
	  	// 注意：此字段仅用于客户端，服务器端不使用。
	  	GetBody func() (io.ReadCloser, error)
	  
	  	// 作用：记录请求体的长度（字节数），-1 表示长度未知。
	  	// 注意：在客户端，一个非空的 Body 和 0 的 ContentLength 也会被视为未知长度，此时会使用分块传输（chunked encoding）。
	  	ContentLength int64
	  
	  	// 作用：列出传输编码，通常用于分块传输。
	  	// 注意：Go 语言会自动处理分块编码，通常无需手动修改此字段。
	  	TransferEncoding []string
	  
	  	// 作用：控制连接在请求-响应周期完成后是否关闭。
	  	// 注意：
	  	// - 在服务器端，此字段由 Go 自动处理，无需手动配置。
	  	// - 客户端将其设为 true 可以防止连接复用。
	  	Close bool
	  
	  	// 作用：指定请求的主机名和端口。
	  	// 注意：
	  	// - 对于服务器端，此字段的值来自 Host 头部。
	  	// - 对于客户端，此字段可选地覆盖 URL.Host 作为发送的 Host 头部。
	  	Host string
	  
	  	// 作用：存储解析后的表单数据，包括 URL 查询参数和 POST/PUT/PATCH 请求体中的数据。
	  	// 注意：此字段仅在调用 ParseForm 或 ParseMultipartForm 后才可用，且仅用于服务器端。
	  	Form url.Values
	  
	  	// 作用：专用于存储从 POST、PUT 或 PATCH 请求体中解析出的表单数据。
	  	// 注意：
	  	// - 不包含 URL 中的查询参数。
	  	// - 仅在调用 ParseForm 后可用，且仅用于服务器端。
	  	PostForm url.Values
	  
	  	// 作用：存储解析后的多部分表单数据，主要用于文件上传。
	  	// 注意：此字段仅在调用 ParseMultipartForm 后才可用，且仅用于服务器端。
	  	MultipartForm *multipart.Form
	  
	  	// 作用：指定在请求体发送后传输的附加头部，用于在流式传输中传递额外信息。
	  	// 注意：此特性并不常用，只有少数 HTTP 客户端和服务器支持。
	  	Trailer Header
	  
	  	// 作用：记录发送请求的远程网络地址（IP:Port）。
	  	// 注意：
	  	// - 此字段由 Go 服务器自动填充，常用于日志记录或安全审计。
	  	// - 客户端请求不使用此字段。
	  	RemoteAddr string
	  
	  	// 作用：保存请求行中未经修改的原始 URI。
	  	// 注意：
	  	// - 通常不推荐直接使用，因为 URL 字段提供了更结构化的访问方式。
	  	// - 此字段仅用于服务器端。
	  	RequestURI string
	  
	  	// 作用：保存请求的 TLS 连接状态信息。
	  	// 注意：仅当连接是加密的（HTTPS）时，Go 服务器才会自动填充此字段。
	  	TLS *tls.ConnectionState
	  
	  	// 作用：一个可选的通道，用于取消客户端请求。
	  	// 注意：此字段已被废弃，现在推荐使用 WithContext() 方法来创建带 Context 的请求，并通过 Context 控制请求的取消。
	  	Cancel <-chan struct{}
	  
	  	// 作用：当发生客户端重定向时，此字段指向导致本次请求的响应。
	  	// 注意：仅在客户端请求重定向时被填充。
	  	Response *Response
	  
	  	// 作用：记录与请求匹配的 ServeMux 模式。
	  	// 注意：当使用 Go 标准库的 ServeMux 路由器时，此字段非空。
	  	Pattern string
	  
	  	// 作用：请求的上下文，用于在整个请求生命周期中传递超时、取消信号和数据。
	  	// 注意：此字段不可直接访问，应通过 req.Context() 获取，并通过 req.WithContext() 来创建包含新上下文的请求副本。
	  	ctx context.Context
	  
	  	// 作用：内部字段，用于存储路由匹配到的模式和通配符参数值。
	  	// 注意：这些字段是 Go 内部使用，不应直接访问。
	  	pat         *pattern
	  	matches     []string
	  	otherValues map[string]string
	  }
	  ```
-