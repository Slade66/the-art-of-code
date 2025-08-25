- **作用：**
	- 将标准 HTTP 请求升级（Upgrade）为持久的全双工 WebSocket 连接。
	- 在完成握手和验证后，它会“劫持”底层的 TCP 连接，并返回一个可用于后续通信的 `*Conn` 对象。
- **方法签名：**
	- ```go
	  func (u *Upgrader) Upgrade(w http.ResponseWriter, r *http.Request, responseHeader http.Header) (*Conn, error)
	  ```
- **参数：**
	- `w http.ResponseWriter`：用于写入 HTTP 响应。
	- `r *http.Request`：这是客户端发来的 HTTP 请求。函数会从该请求中读取各种头部信息，以验证这是否为合法的 WebSocket 握手请求。
	- `responseHeader http.Header`：用于在响应中添加额外的 HTTP 头部。最常见的用途是设置 Cookie。你不能通过它设置 WebSocket 相关的头部，这些头部由库自动处理。
- **返回值：**
	- `*Conn`：握手成功时，函数返回一个 `*websocket.Conn` 指针，表示与客户端建立的 WebSocket 连接。之后所有消息的发送和接收都通过该对象的方法进行。
	- `error`：握手过程中如果出现问题，函数会返回非 `nil` 的 `error`，同时 `*Conn` 为 `nil`。此时，函数已自动向客户端发送相应的 HTTP 错误码。
- **核心执行流程：**
	- **协议头部验证**：严格检查请求头，确保 `Connection: Upgrade`、`Upgrade: websocket`、`GET` 方法和 `Sec-WebSocket-Version: 13` 等握手要素齐全且正确。
	  logseq.order-list-type:: number
	- **安全来源检查**：调用 `Upgrader.CheckOrigin` 函数，验证请求的 `Origin` 头部，以防止跨站 WebSocket 伪造攻击。
	  logseq.order-list-type:: number
	- **密钥质询与计算**：获取客户端的 `Sec-WebSocket-Key`，与一个固定的 GUID 拼接后计算 SHA-1 哈希值，并将其 Base64 编码，为响应做准备。
	  logseq.order-list-type:: number
	- **协商子协议与扩展**：根据客户端请求和服务器配置，确定最终使用的子协议（Subprotocol）和是否启用压缩（Compression）等扩展。
	  logseq.order-list-type:: number
	- **连接劫持（Hijack）**：调用 `http.ResponseWriter` 的 `Hijack` 方法，从 HTTP 服务器接管底层的 TCP 连接控制权。
	  logseq.order-list-type:: number
	- **发送升级成功响应**：向被劫持的连接写入状态码为 `101 Switching Protocols` 的 HTTP 响应，其中包含计算好的 `Sec-WebSocket-Accept` 密钥。
	  logseq.order-list-type:: number
	- **创建并返回连接对象**：用劫持的连接和配置好的缓冲区实例化一个 `*websocket.Conn` 对象，并将其返回给应用程序，用于后续的实时通信。
	  logseq.order-list-type:: number
- **注意：**
	- 记得在使用完成后调用 `func (*Conn) Close` 关闭连接。