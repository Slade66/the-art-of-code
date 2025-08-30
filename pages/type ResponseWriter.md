- **作用：**
	- `ResponseWriter` 接口用于构造并向客户端发送 HTTP 响应，你可以通过它来设置响应头、指定状态码，以及写入响应体。
- **接口定义与方法解析：**
	- ```go
	  type ResponseWriter interface {
	  
	  	// 作用：返回一个 Header 类型，让你能够读写 HTTP 响应头。
	  	// 如何使用：你可以用它来设置各种响应头，例如：
	  	//   - w.Header().Set("Content-Type", "application/json")
	  	//   - w.Header().Set("Cache-Control", "no-cache")
	  	// 注意事项：一旦调用了 WriteHeader 或 Write 方法，大部分响应头就会被发送出去，之后再对 Header 的修改将不再生效。
	  	Header() Header
	  
	  	// 作用：向 HTTP 响应体中写入数据。
	  	// 如何使用：传入一个字节切片 []byte 来作为响应内容，例如：
	  	//   - w.Write([]byte("Hello, World!"))
	  	// 注意事项：
	  	//   - 如果在调用 Write 之前没有显式调用 WriteHeader，第一次调用 Write 时会自动发送 200 OK 状态码。
	  	//   - 如果响应头中没有 Content-Type，Write 会根据你写入的前 512 字节数据自动推断并设置。
	  	Write([]byte) (int, error)
	  
	  	// 作用：设置 HTTP 响应的状态码，例如 200（成功）或 404（未找到）。
	  	// 如何使用：传入一个整型状态码，例如：
	  	//   - w.WriteHeader(http.StatusOK)
	  	//   - w.WriteHeader(http.StatusNotFound)
	  	// 注意事项：
	  	//   - 最好在写入响应体之前设置状态码。
	  	//   - 一旦你调用了 Write，状态码就默认为 200 OK，之后再调用 WriteHeader 将不起作用。
	  	WriteHeader(statusCode int)
	  }
	  ```
-