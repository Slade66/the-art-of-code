- **作用：**
	- 这个函数只负责创建命令，定义好它将在哪个容器里、执行什么操作，但并不会马上运行。
	- 这就等于在命令行把 `docker exec` 指令都打好了，手指还悬在回车键上。
- **方法签名：**
	- ```go
	  func (cli *Client) ContainerExecCreate(ctx context.Context, containerID string, options container.ExecOptions) (container.ExecCreateResponse, error)
	  ```
- **参数：**
	- `ctx context.Context`： [[context]]
	- `containerID string`：目标容器的 ID。
	- `options container.ExecOptions`：[[type ExecOptions]]
- **返回值：**
	- `container.ExecCreateResponse`：
		- 操作成功时返回的结构体。该类型是 `types.IDResponse` 的别名，其内部仅包含一个 `ID` 字段。此 `ID` 是新创建的执行实例的唯一标识符，用于在后续调用 `ContainerExec...` 函数时指定操作目标。
	- `error`：
		- 当执行实例创建失败时，返回非 `nil` 值。表示这个命令连准备运行的资格都没有。常见的失败原因包括：指定的容器不存在、容器未处于运行状态等。