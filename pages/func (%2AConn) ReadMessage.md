- **作用：**
	- 一次性地从 WebSocket 连接中读取一整条完整的消息，并将其内容存入一个字节切片中返回。
- **方法签名：**
	- ```go
	  func (c *Conn) ReadMessage() (messageType int, p []byte, err error)
	  ```
- **参数：**无
- **返回值：**
	- `messageType int`：表示读取到的消息类型。它通常是以下两个常量之一：
		- `websocket.TextMessage`：消息内容是 UTF-8 编码的文本。
		- `websocket.BinaryMessage`：消息内容是二进制数据。
	- `p []byte`：一个字节切片，包含了整条消息的完整内容。
	- `err error`：如果在读取过程中发生任何错误，将返回一个非 `nil` 的 `error`。
- **注意：**
	- **内存占用：**`ReadMessage` 会一次性将整条消息读入内存，如果客户端发送非常大的消息（如大文件），可能会占用大量服务器内存，甚至导致服务崩溃，存在潜在的拒绝服务（DoS）攻击风险。为避免发生此类问题，应在调用 `ReadMessage` 前使用 `conn.SetReadLimit(maxSize)` 设置合理的最大消息体积。如果消息超过限制，`ReadMessage` 会返回错误，从而保护服务器。
	- **阻塞行为：**`ReadMessage` 是阻塞函数，会一直等待直到接收到完整消息或发生错误。因此，应在独立的 Goroutine 中运行读取循环，避免阻塞主程序。
	- **连接关闭：**当连接关闭时，`ReadMessage` 会返回 `*websocket.CloseError` 类型的错误。可以使用 `websocket.IsUnexpectedCloseError()` 判断是否为非预期关闭。