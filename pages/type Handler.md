- **作用：**
	- `Handler` 接口的作用是定义一个处理 HTTP 请求的规范。
	- 任何实现了此接口（`ServeHTTP` 方法）的类型，都可以作为一个 HTTP 请求处理器，被 Go 的 Web 服务器调用。
- **接口定义与方法解析：**
	- ```go
	  type Handler interface {
	  	// 作用：用于处理 HTTP 请求，是 Handler 接口唯一的方法。
	  	// 如何使用：
	  	//    - 第一个参数是 http.ResponseWriter，用于向客户端发送响应。
	  	//    - 第二个参数是 *http.Request，包含了所有请求信息，例如请求方法、URL 和请求体。
	  	// 注意事项：
	  	//    - 线程安全：ServeHTTP 方法可能被并发调用，因此你的实现必须是线程安全的。
	  	ServeHTTP(ResponseWriter, *Request)
	  }
	  ```
-