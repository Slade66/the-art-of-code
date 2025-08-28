-
- [[type Upgrader]]
- [[func (*Upgrader) Upgrade]]
- [[type Conn]]
- [[func (*Conn) ReadMessage]]
- [[func (*Conn) WriteMessage]]
- [[func IsUnexpectedCloseError]]
- **下载：**`go get github.com/gorilla/websocket`
- **两种消息收发方式：**
	- **`ReadMessage` 和 `WriteMessage`：**一次性读写完整消息。适合聊天、通知等场景，简单直接。
	- **`NextReader` 和 `NextWriter`：**流式读写消息。避免一次性载入全部数据到内存，适用于文件传输等大负载场景。
- **怎么用？**
	- **服务器端使用流程：**
		- **创建 HTTP 服务**：启动 HTTP 服务器，并为 WebSocket 路由注册一个处理函数。
		  logseq.order-list-type:: number
		- **实例化升级器**：创建一个 `websocket.Upgrader` 实例，用于处理协议升级请求。
		  logseq.order-list-type:: number
		- **处理握手请求**：在处理函数中调用 `upgrader.Upgrade` 方法，完成握手并获取连接对象。
		  logseq.order-list-type:: number
		- **循环读写消息**：在一个 `for` 循环中，通过 `conn` 对象持续接收和发送消息。
		  logseq.order-list-type:: number
		- **释放连接**：使用 `defer conn.Close()` 语句，确保连接在最后能被妥善关闭。
		  logseq.order-list-type:: number
- **注意：**
	- **一个读 goroutine，一个写 goroutine：**`Conn` 对象仅支持一个并发读取者和一个并发写入者。你可以，也应该分别开启两个 Goroutine，一个专门负责读取，一个专门负责写入。绝对不要在多个 Goroutine 中同时调用 `ReadMessage`，或同时调用 `WriteMessage`。
	- **心跳：**为了防止连接因长时间不活动而被中间设备断开，客户端需要定期向服务器发送 `Ping` 消息，服务端的 `gorilla/websocket` 库会自动处理 `Ping` 消息并返回 `Pong`。
	- **超时检测：**在读取端可以设置 `ReadDeadline`。每次收到 `Pong` 后重置超时时间；若在指定时间内未收到任何消息（包括 `Pong`），`ReadMessage` 会返回超时错误，此时可判定连接已断开。
-