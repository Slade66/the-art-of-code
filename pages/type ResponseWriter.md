- **作用：**
	- `ResponseWriter` 接口是响应写入器，用于向客户端发送 HTTP 响应。你可以通过它来设置响应头、指定状态码，以及写入响应体。
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
- **`ResponseWriter` 的工作原理：**
	- `http.ResponseWriter` 不是一个你填充完后一次性发送的数据结构，它实际上是对底层网络连接的一个实时抽象。你对 `ResponseWriter` 的所有操作，都会直接映射到向底层 TCP 流写入数据的过程。
	- 一个标准的 HTTP 响应是严格按顺序发送的字节序列：状态行 -> 响应头 -> 空行 -> 响应体。
		- 当你调用 `w.Header().Set()` 时，你只是在修改一个尚未发送的内存中的 Header 映射。在这个阶段，没有任何数据被发送。你可以自由地添加、修改或删除头部信息。
		- 当你调用 `w.WriteHeader(code)` 时，服务器会立即将状态行和所有 Header 写入到底层 TCP 连接。这部分数据一旦发送，就无法撤回或修改。
		- 当你调用 `w.Write(body)` 时，如果之前没有调用过 `WriteHeader`，服务器会为了遵守 HTTP 协议，自动先发送默认的 `200 OK` 状态码和所有 Header，然后才开始发送响应体数据。
-