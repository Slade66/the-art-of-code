- 在 Go 语言中，编译的最小单元是包（Package）（也就是文件夹）。
- **编译缓存：GOCACHE**
	- Go 在你的电脑里维护了一个隐秘的文件夹，叫做 `GOCACHE`。这里面存放的不是最终的二进制文件，而是编译过程中生成的中间文件（`.a` 文件）。
	- **编译的缓存命中：**
		- $CacheKey = Hash(源码内容 + 依赖版本 + 编译器版本 + 环境变量 + 编译参数)$
		- Go 编译器在编译每个包之前，都会计算这个 CacheKey（指纹）：
			- **计算指纹：**比如你有一个 `utils` 包。Go 会把 `utils` 里的所有源码、它依赖的其他包的 ID、以及你用的 `GOOS=linux` 这样的参数，全部丢进哈希函数里，算出一串长长的哈希值。
			- **查找缓存：**去 `GOCACHE` 文件夹里找，有没有这就叫这个哈希值的文件？
				- **命中 (Hit)：**找到了！直接把文件拿来用，跳过编译步骤。
				- **未命中 (Miss)：**没找到。老老实实启动编译器，编译这个包，然后把结果存进 `GOCACHE`，名字就是那个哈希值。
		- Go 的编译器非常聪明。如果代码没改，它会直接用缓存的中间文件。
		- 即使代码有变动，Go 也会只编译变动的部分，大大缩短 `go build` 时间。
		- 当你修改了代码再次构建时，Go 智能识别哪些文件没变，直接复用之前的编译结果。
		- 当你执行 `go build` 时，Go 编译器不会每次都傻乎乎地重头开始。它会问自己一个问题：“这个包的代码变了吗？依赖的包变了吗？编译参数变了吗？” 如果都没变，它就直接复用上次的结果。
	- `GOCACHE` 目录结构：
		- **目录分片：**文件名基于哈希值的前两个字符进行分目录存储。
		- **文件后缀：**
			- `-a`：编译后的归档文件（archive）。
			- `-d`：调试信息。
- **下载模块缓存：GOMODCACHE**
	- 所有下载并校验通过的模块被存储在 `GOMODCACHE` 目录（默认为 `$GOPATH/pkg/mod`）。
	- 即使 `go.mod` 变了，它也只会去下载**新增或变更**的那个包，旧的包直接复用。
	- **目录布局：**
		- `cache/download`：存储原始的 `.zip`, `.mod`, `.info` 文件，直接对应模块代理协议的响应。
		- 根目录下存储解压后的源代码。
	- **优化点**：如果不缓存这个，每次都要去 GitHub/Goproxy 下载几百 MB 数据，受网络影响极大。
- **在 Docker 容器中编译 Go 项目**
	- **为什么在 Docker 容器中编译慢？**
		- 很多初学者把 Go 放入 Docker 时，会发现构建速度变慢了。
		- `GOCACHE` 一直在你的硬盘里，越积越多，越跑越快。
		- 每次 `docker build`，如果不做特殊处理，它都是一个**全新的、空的**环境。它没有过去的记忆（缓存）。
		- 如果不做挂载，每次 Jenkins 启动 Docker 容器都是全新的环境，`go build` 会重新下载几百 MB 的包，且无法利用 Go 的增量编译特性。
	- 利用**分层机制**缓存依赖下载 (`go mod download`)，利用**挂载缓存** (`--mount=type=cache`) 缓存编译中间产物。
	- **缓存下载的依赖包：**
		- 避免了重复下载依赖导致的网络 IO。
		- 利用 Docker 的层缓存（Layer Caching）机制可以显著加速镜像构建。
		- ```Dockerfile
		  FROM golang:1.21
		  
		  WORKDIR /app
		  
		  # 1. 先只复制依赖描述文件 (go.mod / go.sum)
		  COPY go.mod go.sum ./
		  
		  # 2. 下载依赖 (go mod download)
		  # 只要 go.mod 没变，Docker 就会直接使用这一层的缓存，跳过下载！
		  # 这样就不用每次都重新下载互联网上的依赖包了。
		  RUN go mod download
		  
		  # 3. 再复制源代码
		  COPY . .
		  
		  # 4.a 最后编译
		  # 这里依然会重新编译代码，但至少依赖包不用重新下载了。
		  # RUN go build -o myapp main.go
		  
		  # 4.b 挂载缓存编译
		  ...
		  ```
	- **挂载缓存：**
		- 避免了重复编译导致的 CPU 消耗。
		- id:: 69396a70-f38e-446e-be7b-e76ecfd8062c
		  ```Dockerfile
		  RUN --mount=type=cache,target=/root/.cache/go-build \
		      go build -o myapp main.go
		  ```
		- “Docker，请你在宿主机上划一块地方专门存 Go 的构建缓存。每次我运行这行命令时，把那块地方挂载到容器里的 `/root/.cache/go-build`。编译完后，产生的新缓存也留在那，下次构建接着用！”
- **加速构建：不从公网下载模块**
	- 方案一（推荐）：搭建私有 Go Proxy。 在内网部署一个 Athens 或者使用 Nexus/Artifactory 的 Go 代理功能。 将 GOPROXY 设置为：GOPROXY="http://your-internal-nexus:8081,direct"。 这样所有公网包会被 Nexus 缓存，第二次构建直接走内网 Nexus。
	- 方案二（Vendor 模式）。 在开发机上执行 go mod vendor，将 vendor/ 目录提交到 Gitea。构建时使用 go build -mod=vendor。这样构建完全不需要网络下载依赖，但会增加 git 仓库体积。
-