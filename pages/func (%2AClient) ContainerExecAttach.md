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
	- `types.HijackedResponse`：操作成功时返回。该对象封装了底层的网络连接及一个带缓冲的读取器，为客户端和 `exec` 进程之间建立了一个可用于实时读写的双向数据流。
	- `error`：操作失败时返回。如果附加过程中出现任何问题（如 `exec ID` 无效、网络中断等），将返回一个非 `nil` 的错误，此时第一个返回值为 `nil`。
- **注意：**
	- 调用方在使用完毕后必须关闭被劫持的连接，否则会导致资源泄漏。