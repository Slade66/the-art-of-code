- **作用：**
	- 该函数用于启动一个已创建的 `exec` 实例，它通过将 Go 程序的 I/O 流（输入、输出、错误）附加到该进程上，从而建立起一个实时的双向连接，适用于交互式命令场景。
- **方法签名：**
	- ```go
	  func (cli *Client) ContainerExecAttach(ctx context.Context, execID string, config container.ExecAttachOptions) (types.HijackedResponse, error)
	  ```
- **参数：**
	- `ctx context.Context`： [[context]]
	- `execID string`：这个 ID 是你之前调用 `ContainerExecCreate` 接口时成功后获取到的。用于告诉 Docker 守护进程，你想要附加到哪一个具体的 `exec` 进程上。
	- `config container.ExecAttachOptions`：是 [[type ExecStartOptions]] 的别名。
- **返回值：**
	- `types.HijackedResponse`：
		- **结构体定义与字段解析：**
			- ```go
			  type HijackedResponse struct {
			  	// 解释: 一个字符串，用于标识通过此连接传输的数据流的媒体类型 (MIME Type)。
			  	//
			  	// 作用: 告知客户端如何正确解析从 Reader 读取的原始二进制流。例如，当值为
			  	//      "application/vnd.docker.raw-stream" 时，它暗示这是一个特殊的多路复用流，
			  	//      可能同时包含标准输出 (STDOUT) 和标准错误 (STDERR)，需要使用像
			  	//      `stdcopy.StdCopy` 这样的工具来正确分离。
			  	mediaType string
			  
			  	// 解释: Go 语言标准库中的网络连接接口，代表了从 HTTP 协议中“劫持”而来的
			  	//      最底层的、原始的双向 TCP 连接。
			  	//
			  	// 作用: 作为双向数据流中的【写入端】。其核心职责是让客户端能向容器内的进程
			  	//      发送数据（即标准输入 STDIN）。例如，在交互式 Shell 中，你输入的
			  	//      每一个字符或命令，都是通过调用 `Conn.Write(...)` 方法发送过去的。
			  	//
			  	// 示例:
			  	//      // 向容器内的 Shell 发送 "ls -l" 命令并换行
			  	//      hijackedResponse.Conn.Write([]byte("ls -l\n"))
			  	Conn net.Conn
			  
			  	// 解释: 一个指向 Go 标准库 `bufio` 包中读取器的指针，它封装了底层的 `net.Conn`
			  	//      连接，为其提供了高效的缓冲 (buffering) 功能。
			  	//
			  	// 作用: 作为双向数据流中的【读取端】。它负责高效地从容器进程中接收数据流
			  	//      （即标准输出 STDOUT 和标准错误 STDERR）。缓冲机制可以显著提升读取性能，
			  	//      因为它会一次性从网络中读取一大块数据到内存，然后程序可以按需从内存中
			  	//      快速读取，从而减少了频繁的网络 I/O 操作。
			  	//
			  	// 示例:
			  	//      // 使用 stdcopy 将 Reader 中的 STDOUT 和 STDERR 分离并打印到标准输出和标准错误
			  	//      stdcopy.StdCopy(os.Stdout, os.Stderr, hijackedResponse.Reader)
			  	Reader *bufio.Reader
			  }
			  ```
		- **为什么要叫“劫持（Hijack）”？**
			- 客户端会发起一个特殊的升级请求。若服务器同意，双方将共同抛开标准的 HTTP 协议，直接夺取对底层 TCP 网络连接的直接控制权。
			- 这一行为将改变连接的原有用途：它不再是为了一次性的“请求-响应”而建立的短暂通道，而是转变为一个持久的双向数据流。如同建立了一通可以随时实时通话的电话，直至任意一方主动“挂断”。
			- 在这个过程中，HTTP 协议的角色仅相当于一个“引荐人”，在完成牵线搭桥后便不再介入，将控制权完全交给了应用程序。
			- 所以，“劫持”这个词恰如其分地描述了这种“跳出协议常规，接管底层通道控制权”的行为。这实际上是一种双方事先约定的“友好劫持”，目的是为了实现标准 HTTP 难以做到的实时、交互式通信。
	- `error`：操作失败时返回。如果附加过程中出现任何问题（如 `exec ID` 无效、网络中断等），将返回一个非 `nil` 的错误，此时第一个返回值为 `nil`。
- **注意：**
	- 调用方在使用完被劫持的连接后，必须关闭它（`hijackedResponse.Conn.Close()`），否则会造成资源泄漏。