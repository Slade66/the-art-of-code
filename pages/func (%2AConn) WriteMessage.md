- **作用：**
	- 将一个完整的字节切片作为一条独立的消息，一次性地写入到 WebSocket 连接中。
- **方法签名：**
	- ```go
	  func (c *Conn) WriteMessage(messageType int, data []byte) error
	  ```
- **参数：**
	- `messageType int`：指定要发送的消息类型。
	- `data []byte`：一个字节切片，包含了你想要发送的整条消息的完整内容。
- **返回值：**
	- `error`：如果在写入过程中发生任何错误（例如连接已关闭、网络问题、写入超时等），函数将返回一个非 `nil` 的 `error`。如果消息成功写入，则返回 `nil`。
- **注意：**
	- **并发安全：**`WriteMessage` 不是并发安全的，必须保证同一时间只有一个 goroutines 调用该函数。