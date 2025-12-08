- **作用：**
	- 该函数用于查看 `exec` 任务的运行状态。
- **方法签名：**
	- ```go
	  func (cli *Client) ContainerExecInspect(ctx context.Context, execID string) (container.ExecInspect, error)
	  ```
- **参数：**
	- `ctx context.Context`：[[Go 程序设计语言/context]]
	- `execID string`：你想要查询的那个 `exec` 任务的 ID。
- **返回值：**
	- `container.ExecInspect`：这个结构体包含了 `exec` 任务的运行状态信息。
		- ```go
		  // ExecInspect holds information returned by exec inspect.
		  type ExecInspect struct {
		  	// 解释: Docker 为这个 exec 命令分配的唯一 ID。
		  	// 作用: 用这个 ID 告诉 Docker 你要操作的是哪一个 exec 命令。
		  	// 示例: "b8e3a5a4d2e1f..."
		  	ExecID string `json:"ID"`
		  
		  	// 解释: 这个命令在哪个容器里运行。
		  	// 作用: 让你知道这个 exec 属于哪个容器。
		  	// 示例: "a7b3e4c5f6a1d..."
		  	ContainerID string
		  
		  	// 解释: true 表示命令还在跑，false 表示已经停了。
		  	// 作用: 检查这个值就能知道命令执行完了没有。
		  	// 示例: true (正在执行), false (已结束)
		  	Running bool
		  
		  	// 解释: 命令结束后，告诉我们结果怎么样的一个数字。
		  	// 作用: 看这个数字是不是 0，是 0 就代表成功，不是 0 就代表失败。
		  	// 示例: 0 (成功), 1 (失败)
		  	ExitCode int
		  
		  	// 解释: 这个命令在你的物理机上的进程编号。
		  	// 作用: 如果需要用物理上的一些工具去监控或调试这个命令，就用这个编号。
		  	// 示例: 24581
		  	Pid int
		  }
		  ```
	- `error`：如果在查询过程中发生任何错误（如网络问题、`execID` 不存在），将返回一个非 `nil` 的 `error` 对象。
- **注意：**
	- 该函数用于检查任务的状态，例如任务是否仍在运行、其最终的退出码是多少，但它并不负责捕获命令执行后产生的输出。