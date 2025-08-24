- **作用：**用于创建一个可以与 Docker 守护进程通信的 API 客户端。
- **方法签名：**
	- ```go
	  func NewClientWithOpts(ops ...Opt) (*Client, error)
	  ```
- **参数：**
	- `ops ...Opt`：
		- `Opt` 是函数类型：`type Opt func(*Client) error`，接收一个 `*Client` 指针并返回 `error`。
		- `...` 表示可变参数，可传入零个、一个或多个 `Opt`。
		- 调用者可以通过传入一系列选项函数来覆盖或修改客户端的默认行为。
- **返回值：**
	- 成功时返回 `*Client` 和 `nil`。
	- 失败时返回 `nil` 和相应的 `error`。
- **核心执行流程：**
	- **初始化默认实例：**函数首先根据内置常量创建一个包含默认 `http.Client` 的基础客户端实例。
	  logseq.order-list-type:: number
	- **应用自定义选项：**随后，函数依次遍历传入的选项函数，并将客户端实例指针传入，使其能够直接修改实例，以覆盖默认配置或增强功能。
	  logseq.order-list-type:: number
	- **封装并返回：**在所有选项应用完毕后，函数会进行必要的收尾操作（如添加链路追踪功能），并返回最终配置好的客户端实例；若过程中出现错误，则中止并返回该错误。
	  logseq.order-list-type:: number
- **注意：**
	- 创建 Docker 客户端时应始终使用 `NewClientWithOpts`，因为 `NewClient` 和 `NewEnvClient` 已被弃用。
		- `NewClient`：旧版函数，参数固定，缺乏灵活性。
		- `NewEnvClient`：通过环境变量创建客户端，现在可用 `NewClientWithOpts(client.FromEnv)` 替代。