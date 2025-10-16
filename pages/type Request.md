- **作用：**
	- `Request` 结构体封装了从客户端发来的 HTTP 请求的全部信息。
	- 后端开发的工作就是从这个结构体中提取信息并据此作出响应。
- **结构体定义与字段解析：**
	- ```go
	  type Request struct {
	  	// Method 指定 HTTP 请求的方法，例如 "GET", "POST" 等。
	  	// - 作用：定义请求的类型，是路由和处理的基础。
	  	// - 注意：对于客户端请求，如果此字段为空，默认使用 "GET" 方法。
	  	Method string
	  
	  	// URL 代表请求的 URL，包含了请求的路径和查询参数等。
	  	// - 作用：是整个请求的核心部分，通过它可以获取请求的路径、查询参数等信息。
	  	// - 注意：使用 r.URL.Path 获取请求的路径部分；使用 r.URL.Query() 获取查询字符串，并通过 Get() 方法获取参数值。
	  	URL *url.URL
	  
	  	// Header 存储请求的 HTTP 头部，包含各种元数据。
	  	// - 作用：包含请求的元数据，如 Content-Type、Authorization 等。
	  	// - 注意：HTTP 头的键是大小写不敏感的，应使用 Get() 方法获取头信息，该方法会自动处理规范化。
	  	Header Header
	  
	  	// Body 承载 POST、PUT 等请求的主体数据。
	  	// - 作用：用于传递请求的主体内容，如 JSON 或表单数据。
	  	// - 注意：Body 是一个数据流，只能被读取一次，读取后必须调用 Close() 来释放资源。
	  	Body io.ReadCloser
	  
	  	// Host 指定请求的主机名和端口。
	  	// - 作用：通常来自请求头中的 Host 字段。
	  	// - 注意：对于服务器端，此字段的值来自 Host 头部。
	  	Host string
	  
	  	// Form 存储解析后的所有表单数据，包括 URL 查询参数和请求体数据。
	  	// - 作用：方便地访问所有表单数据。
	  	// - 注意：此字段仅在调用 ParseForm() 或 ParseMultipartForm() 后才可用。
	  	Form url.Values
	  
	  	// PostForm 仅存储从请求体中解析出的表单数据。
	  	// - 作用：专门用于获取 POST/PUT/PATCH 请求体中的数据。
	  	// - 注意：此字段仅在调用 ParseForm() 后可用，且不包含 URL 查询参数。
	  	PostForm url.Values
	  
	  	// RemoteAddr 记录发送请求的远程网络地址（IP:Port）。
	  	// - 作用：常用于日志记录或安全审计。
	  	// - 注意：此字段由 Go 服务器自动填充，客户端请求不使用。
	  	RemoteAddr string
	  
	  	// Context 承载请求的上下文，用于传递超时、取消信号和数据。
	  	// - 作用：控制请求的生命周期和数据传递。
	  	// - 注意：此字段不能直接访问，应通过 req.Context() 获取，并通过 req.WithContext() 创建副本。
	  	ctx context.Context
	  }
	  ```
-