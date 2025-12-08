- **作用：**启动一个先前已经在容器内部创建好的执行实例。
- **方法签名：**
	- ```go
	  func (cli *Client) ContainerExecStart(ctx context.Context, execID string, config container.ExecStartOptions) error
	  ```
- **参数：**
	- `ctx context.Context`： [[Go 程序设计语言/context]]
	- `execID string`：执行实例的唯一标识符。它是由上一步 `ContainerExecCreate` 函数成功调用后返回的。
	- `config container.ExecStartOptions`：[[type ExecStartOptions]]
- **返回值：**
	- `error`：如果函数执行成功，它将返回 `nil`；如果中间发生任何错误（如网络问题、`execID` 无效等），它将返回一个非空的 `error` 对象。
- **注意：**
	- 函数成功返回 `nil` 仅代表启动操作完成，而命令的最终执行结果（如退出码）必须通过 `ContainerExecInspect` 进行异步查询。