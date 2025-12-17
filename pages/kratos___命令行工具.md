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
		- **生成的代码：**
			- ```proto
			  syntax = "proto3";
			  
			  package api.apisix.v1;
			  
			  option go_package = "user-service/api/apisix/v1;v1";
			  option java_multiple_files = true;
			  option java_package = "api.apisix.v1";
			  
			  service Apisix {
			  	rpc CreateApisix (CreateApisixRequest) returns (CreateApisixReply);
			  	rpc UpdateApisix (UpdateApisixRequest) returns (UpdateApisixReply);
			  	rpc DeleteApisix (DeleteApisixRequest) returns (DeleteApisixReply);
			  	rpc GetApisix (GetApisixRequest) returns (GetApisixReply);
			  	rpc ListApisix (ListApisixRequest) returns (ListApisixReply);
			  }
			  
			  message CreateApisixRequest {}
			  message CreateApisixReply {}
			  
			  message UpdateApisixRequest {}
			  message UpdateApisixReply {}
			  
			  message DeleteApisixRequest {}
			  message DeleteApisixReply {}
			  
			  message GetApisixRequest {}
			  message GetApisixReply {}
			  
			  message ListApisixRequest {}
			  message ListApisixReply {}
			  ```
- **通过 proto 文件生成 Service 模板代码：**
	- `kratos proto server` 命令用于根据 `.proto` 文件生成服务端代码的模板，包括接口定义和方法的空实现，便于你在其中编写具体的业务逻辑。
	- `-t` 参数（target）用于指定生成代码的目标目录，Kratos 会将模板文件输出到该目录下。
	- **举例：**
		- ```bash
		  kratos proto server api/bubble/v1/todo.proto -t internal/service
		  ```
		- Kratos 会根据 `.proto` 中定义的服务名称，在 `internal/service` 目录下生成对应的 `.go` 文件（如 `todo.go`），其中包含服务方法的空实现，供你后续填充业务逻辑。