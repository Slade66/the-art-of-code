- **作用：**
	- 调整一个正在容器内执行的命令所关联的 TTY 终端的尺寸。
- **方法签名：**
	- ```go
	  func (cli *Client) ContainerExecResize(ctx context.Context, execID string, options container.ResizeOptions) error
	  ```
- **参数：**
	- `ctx context.Context`： [[context]]
	- `execID string`：这是要调整大小的执行实例的唯一标识符。这个 ID 由 `ContainerExecCreate` 函数返回。
	- `options container.ResizeOptions`：
		- ```go
		  type ResizeOptions struct {
		      Height uint // 新的高度（行数）
		      Width  uint // 新的宽度（列数）
		  }
		  ```
- **返回值：**
	- `error`：如果函数执行过程中出现问题（比如网络错误、`execID` 不存在等），它会返回一个非 `nil` 的 `error` 对象。如果一切顺利，它将返回 `nil`。