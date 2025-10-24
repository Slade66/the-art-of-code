- **什么是 Docker 镜像（Image）？**
	- Docker 镜像就像在虚拟机中安装好操作系统并配置好服务后生成的系统快照，用作创建容器的模板。
	- 它定义了容器运行所需的操作系统和应用环境，是构建容器的起点。
- Docker 命令
  heading:: true
	- 常用命令
	  heading:: true
		- `docker run`
			- 从镜像创建容器并立即运行。
			- **语法：**
				- ```bash
				  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
				  ```
			- `OPTIONS`
				- 可选参数，用于控制容器的运行行为。
				- `-d`：使容器在后台运行，不占用当前终端。
				- `--name <容器名>`：为容器指定名称，方便后续通过名称进行管理。
				- `-p <主机端口>:<容器端口>`：端口映射，将容器内的端口暴露给主机，从而通过主机端口访问容器中的服务。
				- `-v <本机目录>:<容器目录>`：目录挂载，将主机目录挂载到容器中，实现宿主机与容器之间的文件共享。
				- `--network <网络名>`：指定容器连接到的 Docker 网络，便于容器之间通信。
				- `-it`：默认情况下，容器启动后执行镜像中的命令，命令执行完毕容器即退出。加上 `-it` 后，容器将以交互模式运行，允许你像登录一台 Linux 主机一样进入容器，手动输入命令并实时查看输出。
				- `--rm`：容器退出后自动删除，适用于临时任务或测试场景，可避免系统中残留无用容器，相当于“用完即扔”。
			- `IMAGE`
				- 指定你希望运行的镜像。
				- 如果本地不存在该镜像，Docker 会自动从默认仓库 Docker Hub 下载。
- 踩坑经验
  heading:: true
	- [[连接不上 Docker 中的 MySQL 服务]]
- [[Docker 的仓库管理]]
- [[Docker 的镜像管理]]
- [[容器（Container）]]
- [[Docker Desktop]]
- [[Docker/Docker Engine Go SDK]]
- docker login
  collapsed:: true
	- 作用：让你登录到一个指定的 Docker 镜像仓库（Registry），从而获得推送（push）和拉取（pull）私有镜像的权限。
	- 镜像仓库就像存放代码的仓库（比如 GitHub），不过它存放的是打包好的 Docker 镜像。最常见的镜像仓库：Docker Hub，它是 Docker 官方的公共仓库。
	- **拉取公共镜像**：通常不需要登录，任何人都可以匿名拉取。
	- **推送任何镜像**：**必须登录**。你总得告诉仓库你是谁，有没有权限把镜像上传到这个地方。
	- **拉取私有镜像**：**必须登录**。私有镜像是为了保护代码和应用，只有经过授权的用户才能拉取。
	- 你工作中 99% 的场景都是将你的应用（比如你用 Kratos 写的微服务）打包成镜像，然后推送到公司的**私有仓库**中，再由 Kubernetes 或其他部署系统从私有仓库拉取镜像来运行。这个过程的第一步就是 `docker login`。
	- 登录 Docker Hub
		- 省略 `SERVER` 则登录 Docker Hub。
		- docker login
		- 执行后，终端会提示你输入用户名和密码：
	- 登录私有仓库
		- docker login my-registry.company.com:5000
	- 登录信息存放在哪里？
		- 当你成功执行 `docker login` 后，Docker 会将认证信息（一个经过编码的授权令牌）保存在你用户主目录下的一个文件里：
		- **文件路径**: `~/.docker/config.json`
		- 它是一个 JSON 格式的文件，`auths` 字段下记录了你登录过的所有仓库地址和对应的授权令牌。
- `docker logout`
  collapsed:: true
	- 如果你想清除登录凭据，可以使用 `docker logout` 命令
	- 登出 Docker Hub
	  docker logout
	- 登出指定的私有仓库
	  docker logout my-registry.company.com:5000
- 当用户手动勾选“跳过验证”时，是用户在说：“我确认 10.30.60.149 这个地址是我内网里绝对可信的服务器，我愿意为这个连接承担潜在的风险。”
- customTransport := &http.Transport{
      8         TLSClientConfig: &tls.Config{InsecureSkipVerify: true},
      9     }
- [[Docker/host.docker.internal]]
- **Docker API 版本不匹配问题**
  collapsed:: true
	- **问题原因：**Docker 客户端与 Docker daemon 通过 REST API 进行通信，API 版本随着 Docker 更新而变化。Go 应用中的 `github.com/docker/docker/client` 库默认使用 API 版本 1.51，而本地 Docker daemon（如 Docker Desktop）可能只支持最高版本 1.49。若客户端尝试使用比 daemon 支持的更高版本，daemon 会拒绝连接并返回错误。
	- **解决方法**：通过启用 Docker SDK 的 API 版本协商协议，客户端会自动与 Docker daemon 协商并使用兼容的 API 版本，从而避免版本不匹配问题。
	- **示例代码：**
		- ```go
		  cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
		  ```
- **配置不安全的 Registry**
  collapsed:: true
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
  collapsed:: true
	- 当 Go 应用调用 `cli.RegistryLogin(ctx, authConfig)` 时，它向本地 Docker daemon 发送 API 请求，由 Docker daemon 实际完成与 Docker Registry 的网络通信和认证。
-