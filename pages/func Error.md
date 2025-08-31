- **作用：**用于快速向客户端返回一个格式规范的 HTTP 错误响应。
- **方法签名：**
	- ```go
	  func Error(w ResponseWriter, error string, code int)
	  ```
- **参数：**
	- **`w ResponseWriter`：**[[type ResponseWriter]]
	- **`error string`：**错误消息，该字符串会作为纯文本的响应体发送给客户端。
	- **`code int`：**HTTP 状态码，例如 `http.StatusNotFound` 或 `http.StatusInternalServerError`。
- **返回值：**无。它直接对传入的 `http.ResponseWriter` 进行操作，将错误响应写入其中。
- **核心执行流程：**
	- **设置 HTTP 头部：**函数会强制将 `Content-Type` 设为 `text/plain`，并添加 `X-Content-Type-Options: nosniff` 头部，同时删除可能存在的 `Content-Length` 头部，以确保响应头部的正确性。
	- **写入状态码：**它调用 `w.WriteHeader`，将你传入的 HTTP 状态码写入响应。
	- **写入错误消息：**它将你提供的错误消息作为纯文本响应体写入。
- **注意：**
	- **非终止性：**`http.Error` 不会终止请求处理。因此，在调用该函数后，你必须紧接着使用 `return` 语句，以确保你的处理函数不再继续执行，从而避免向响应中写入额外的数据。