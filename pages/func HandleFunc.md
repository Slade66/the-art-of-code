- **作用：**
	- 用于注册一个处理器函数。将一个特定的 URL 路径（pattern）和一个处理该路径请求的函数（handler）关联起来。当服务器接收到与该路径匹配的 HTTP 请求时，它会调用你注册的这个函数来处理。
	- 简单来说，这个函数就是告诉 Go 程序：“当有用户访问这个地址时，就调用这个函数去处理。”
	  id:: 68b2bfc3-c4e1-47ff-95c4-ad6ac0483a97
- **方法签名：**
	- ```go
	  func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
	  ```
- **参数：**
	- **`pattern string`：**你想要匹配的 HTTP 请求的路径。
	- **`handler func(ResponseWriter, *Request)`：**HTTP 请求的处理器函数。它决定了当一个请求与 `pattern` 匹配时，服务器会做什么。这个函数接收两个参数：
		- [[type ResponseWriter]]
		- [[type Request]]
- **返回值：**
	- 无。
- **注意：**
	- **全局单例：**`http.HandleFunc` 会把处理器注册到全局的 `http.DefaultServeMux` 上。这意味着所有 HTTP 服务器实例都共享同一套路由，无法独立配置各自的路由规则。
	- **路由冲突：**如果你多次为同一个路径（`pattern`）调用 `http.HandleFunc`，后注册的处理器会覆盖掉之前注册的处理器。
-