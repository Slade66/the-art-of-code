Kratos 项目的目录结构
heading:: true
	- 想写好 Kratos 项目，首先需要理解它的代码是如何组织的。
	- Kratos 的目录结构遵循官方推荐的标准化工程架构，清晰且便于维护。
	- `api`：
		- 用于存放 proto 文件的目录。在这里定义服务接口（`service`）和消息格式（`message`）。
		- **为什么会有 v1 文件夹？**
			- 前端和后端通过接口（API）进行数据交换。
			- 假设你已经上线了某个接口，且前端正在使用。如果后续需要修改接口，直接修改旧接口可能导致前端代码出错，因为前端仍依赖于旧的接口定义。为避免这种情况，我们通常使用接口版本控制。即在接口发生变化时，为其分配新版本号，前端可以选择是否升级到新版本，而旧版本接口仍然可用，不会影响现有功能。
			- `v1` 表示接口的第一个版本（Version 1）。如果以后需要修改接口，可以创建 `v2` 文件夹，表示接口的第二个版本。
			- 在项目初期，直接将 Proto 文件放在 `api` 文件夹下没有问题，因为此时只有一个版本。
			- 当接口发生不兼容的变化时，你可以创建 `v1` 或 `v2` 文件夹来进行版本管理。
	- `bin`：用于存放编译生成的可执行文件。
	- `cmd`：
		- 用于放置 Kratos 项目中每个服务的启动入口（main 函数）。
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
	- **添加 proto 文件：**
		- `kratos proto add` 命令用于将 proto 文件快速添加到项目的指定目录中。
		- **举例：**
			- ```bash
			   kratos proto add api/bubble/v1/todo.proto
			  ```
			- 执行该命令后，`api/bubble/v1/todo.proto` 文件会被添加到 Kratos 项目的 `api/bubble/v1` 目录。如果该目录不存在，命令会自动创建所需的目录结构。
			- 同时，命令还会自动生成服务接口和数据结构的代码，帮助你快速开发。
	- **根据 proto 文件生成客户端代码：**
		- `kratos proto client` 命令用于根据 `.proto` 文件生成对应的 Go 语言客户端代码。
		- **举例：**
			- ```bash
			  kratos proto client api/bubble/v1/todo.proto
			  ```
			- **会生成以下文件：**
				- `.pb.go`：包含消息类型和服务接口的 Go 实现及其序列化方法；
				- `_grpc.pb.go`：包含客户端可调用的接口定义，以及服务端需要实现的接口；
				- `_http.pb.go`：仅当 `.proto` 文件中声明了 HTTP 映射（`google.api.http`）时才会生成，包含 HTTP 路由相关代码。
	- **通过 proto 文件生成 Service 模板代码：**
		- `kratos proto server` 命令用于根据 `.proto` 文件生成服务端代码的模板，包括接口定义和方法的空实现，便于你在其中编写具体的业务逻辑。
		- `-t` 参数（target）用于指定生成代码的目标目录，Kratos 会将模板文件输出到该目录下。
		- **举例：**
			- ```bash
			  kratos proto server api/bubble/v1/todo.proto -t internal/service
			  ```
			- Kratos 会根据 `.proto` 中定义的服务名称，在 `internal/service` 目录下生成对应的 `.go` 文件（如 `todo.go`），其中包含服务方法的空实现，供你后续填充业务逻辑。
- 如何编译 Kratos 项目
  heading:: true
-