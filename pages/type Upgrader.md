- **作用：**
	- WebSocket 连接最初由一个标准的 HTTP 请求发起，请求中包含 `Upgrade: websocket` 头部，用来表明客户端希望将连接升级为 WebSocket。一旦服务器同意并返回 `101 Switching Protocols` 状态码，底层 TCP 连接就会切换为 WebSocket 协议，开始双向通信。
	- `Upgrader` 用于将 HTTP 连接升级为 WebSocket，并允许配置升级过程中的相关参数。
- **结构体定义与字段解析：**
	- ```go
	  type Upgrader struct {
	      // 作用：
	  	// ReadBufferSize 指定了用于从网络连接读取数据的 I/O 缓冲区大小（以字节为单位）。
	  	// 这是一个性能优化字段。它通过减少底层网络读取的系统调用次数来提高效率。库会尝试一次性从网络中读取最多 ReadBufferSize 字节的数据放入内存缓冲区。
	  	// 注意：
	  	// 1. 此大小不限制单条消息的最大体积。
	  	// 2. 如果设置为 0，将使用 Go HTTP 服务器默认的缓冲区（通常是 4096 字节）。
	  	// 3. 在处理大量频繁的小消息时，合理调整此值可以显著影响性能和内存占用。
	  	ReadBufferSize int
	  
	    	// 作用：
	  	// WriteBufferSize 指定了用于向网络连接写入数据的 I/O 缓冲区大小（以字节为单位）。
	  	// 与 ReadBufferSize 类似，这也是一个性能优化字段。发送的数据会先暂存在此缓冲区，然后一次性写入网络，以减少系统调用。
	  	// 注意：
	  	// 1. 此大小不限制单条消息的最大体积。
	  	// 2. 如果设置为 0，将使用 Go HTTP 服务器默认的缓冲区。
	  	// 3. 适当增大此值可以将在短时间内发送的多个小消息“打包”发送，降低 CPU 负载。
	  	WriteBufferSize int
	  
	    	// 作用：
	  	// CheckOrigin 是一个回调函数，返回布尔值，用于验证 WebSocket 请求的来源（Origin）是否可接受。
	  	// 这是最关键的安全配置，用于防止跨站 WebSocket 伪造攻击。浏览器发起的请求会携带 Origin HTTP 头部，此函数会对其进行校验。
	  	// 注意：
	  	// 1. 在生产环境中，【必须】实现此函数，严格校验请求来源是否为你的可信域名。
	  	// 2. 在开发阶段，为方便调试，可以暂时设置为 `func(r *http.Request) bool { return true }` 来允许所有来源。
	  	// 3. 如果此字段为 nil，库会启用一个安全的默认策略：拒绝所有跨域请求（即要求 Origin 头部与 Host 头部匹配）。
	  	CheckOrigin func(r *http.Request) bool
	  }
	  ```
-