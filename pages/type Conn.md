- **作用：**
	- `Conn` 类型表示一个 WebSocket 连接。在 HTTP 请求处理函数中，通过调用 `Upgrader.Upgrade` 方法，可以将 HTTP 连接升级为 WebSocket 连接。升级成功后，该方法会返回一个 `*websocket.Conn` 对象，它代表一个持久的全双工通信通道，之后所有与客户端的消息收发都通过这个对象完成。
- **结构体定义与字段解析：**
	- ```go
	  用不到，略
	  ```