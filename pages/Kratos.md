Kratos 项目的目录结构
heading:: true
	- 想写好 Kratos 项目，首先需要理解它的代码是如何组织的。
	- Kratos 的目录结构遵循官方推荐的标准化工程架构，清晰且便于维护。
	- `api`：用于存放使用 Protocol Buffers 定义的服务接口（`service`）和数据结构（`message`）。
	- `bin`：用于存放编译生成的可执行文件。
	- `cmd`：
		- 用于放置 Kratos 项目中每个服务的启动入口（main 函数）
		- 只负责应用的启动逻辑，业务逻辑应放在 `internal/` 中实现。
		- 在一个项目中，可以包含多个子程序。每个 `cmd/<子目录>` 对应一个可独立构建和运行的可执行程序。通常一个微服务对应一个子目录，每个目录下都包含一个 `main.go` 文件（如 `cmd/xxx/main.go`），作为该服务的启动入口。
	- `configs`：存放项目的配置文件（如 `config.yaml`）。
	- `internal`：存放核心的业务逻辑代码。
		- `biz`：负责核心业务逻辑，比如处理用户注册、订单计算等，只关注业务本身，不管数据从哪来。
		- `data`：实现数据的读写操作，比如查数据库、调外部接口等，给 `biz` 提供数据支持。
		- `service`：接收 gRPC 或 HTTP 的请求，做简单的参数处理，然后调用 `biz` 执行业务逻辑。
		- `server`：负责启动服务，把 `service` 绑定到 gRPC 或 HTTP 接口上，让服务真正运行起来。
		- `conf`：定义配置用的结构体，用来加载和使用配置文件里的内容（比如端口号、数据库地址等）。
	- `third_party`：用于存放所有外部依赖的 `.proto` 文件。
- kratos 命令行工具
  heading:: true
	- **安装：**`go install github.com/go-kratos/kratos/cmd/kratos/v2@latest`
	- **创建项目：**
		- 使用默认模板创建一个新的 Kratos 项目：`kratos new <项目名>`
		- Kratos 通过 Git 仓库管理项目模板，创建项目时会自动拉取模板并初始化项目目录结构。
		- 使用 `-r` 参数可以指定自定义模板仓库来创建项目。
		- 使用 `--nomod` 参数添加服务：告诉 Kratos 在创建服务时，不要生成独立的模块（即不创建新的 `go.mod` 文件），而是将新服务作为当前项目的一部分，使用统一的依赖管理（共用项目根目录下的 `go.mod`）。
		- 如果遇到网络拉取失败的问题（如 GitHub 访问受限），可以设置代理，例如：
			- ```shell
			  $env:HTTPS_PROXY = "http://127.0.0.1:10809"
			  $env:HTTP_PROXY = "http://127.0.0.1:10809"
			  ```
- 如何编译 Kratos 项目
  heading:: true
-