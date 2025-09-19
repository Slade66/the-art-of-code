- [[容器（Container）]]
- [[Docker Desktop]]
- [[Docker Engine Go SDK]]
- docker login
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
-