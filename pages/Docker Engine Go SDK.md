- **Docker API 版本不匹配问题**
	- **问题原因：**Docker 客户端与 Docker daemon 通过 REST API 进行通信，API 版本随着 Docker 更新而变化。Go 应用中的 `github.com/docker/docker/client` 库默认使用 API 版本 1.51，而本地 Docker daemon（如 Docker Desktop）可能只支持最高版本 1.49。若客户端尝试使用比 daemon 支持的更高版本，daemon 会拒绝连接并返回错误。
	- **解决方法**：通过启用 Docker SDK 的 API 版本协商协议，客户端会自动与 Docker daemon 协商并使用兼容的 API 版本，从而避免版本不匹配问题。
	- **示例代码：**
		- ```go
		  cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
		  ```
- **配置不安全的 Registry**
	- 如果你的 Registry 在一个信任的内部网络中，并且你愿意降低一些安全性来让配置更简单，可以把它设置为“不安全 Registry”，也就是让 Docker 连接时不检查 TLS 证书。
	- **Docker Desktop 配置步骤：**
		- 打开 Docker Desktop 设置，右键点击任务栏的 Docker 图标，选择 "Settings"。
		  logseq.order-list-type:: number
		- 导航至 "Docker Engine"
		  logseq.order-list-type:: number
		- 在 `daemon.json` 中，添加 `insecure-registries` 字段并填写 Registry 地址：
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```json
			  {
			    "insecure-registries": ["10.30.60.149:5000"],
			  }
			  ```
		- 点击 "Apply & Restart" 应用并重启 Docker。
		  logseq.order-list-type:: number
- **登录 Registry 的流程**
	- 当 Go 应用调用 `cli.RegistryLogin(ctx, authConfig)` 时，它向本地 Docker daemon 发送 API 请求，由 Docker daemon 实际完成与 Docker Registry 的网络通信和认证。
-