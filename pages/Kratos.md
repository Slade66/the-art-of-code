- [[Kratos/config]]
- [[Kratos/registry]]
- [[Kratos/错误处理]]
- Kratos 的概念
  heading:: true
  collapsed:: true
	- **API 优先的设计理念：**
		- **核心思想：**“API 优先”强调在编写任何具体业务逻辑代码之前，首先设计应用程序接口（API）。这意味着我们首先要用一种与语言无关的方式来清晰地定义服务能提供哪些功能、需要什么数据、返回什么数据。
		- **流程：**设计 API（`.proto` 文件）→ 生成代码 → 实现业务逻辑。
		- API 定义独立于代码，是首要的设计产物，而代码则是对该定义的“实现”。
		- 相比之下，Spring Boot 采用的是“代码优先”的方式，流程为：编写业务代码（Java Class）→ 通过注解定义 API → 生成 API 文档。在这种方式中，代码实现是核心，API 则是代码的“副产品”，通过注解附加到代码上，API 文档是从代码中反向生成的。
		- 这种 API 定义如同具有法律效力的“合同”，采用与特定编程语言无关的格式描述，如 Kratos 使用的 Protobuf，或 Web 开发中常见的 OpenAPI（Swagger）。
		- 这份“合同”明确规定了：
			- 服务能提供哪些功能（RPC 方法）。
			- 调用这些功能需要什么样的数据（请求消息）。
			- 功能执行后返回的数据（响应消息）。
			- 可能出现的错误类型和错误码。
		- 只有当这份“合同”经过团队（前端、后端）共同评审并确认后，后端工程师才会开始实现其中承诺的功能，前端工程师则可基于该合同进行界面开发，而无需等待后端代码完成。
		- 最直接的好处是促进团队并行开发。API “合同”确定后，后端和前端可以同时进行工作。前端工程师可以使用工具根据 API 定义模拟一个假的服务端（Mock Server）来进行开发和调试，从而大大缩短项目周期。
		- 这种方式还可降低沟通成本和集成风险：API “合同”是所有团队成员共同遵循的唯一标准，避免了口头约定或模糊文档带来的误解。最终进行前后端联调时，由于双方都严格遵守相同的合同，集成过程通常非常顺利，有效避免了“你传的参数不对”和“我返回的格式不是这样的”等常见问题。
		- 首先，在 `.proto` 文件中使用 Protobuf 接口定义语言定义服务接口、消息数据结构和错误码。然后，通过工具生成 gRPC 和 HTTP 的服务端/客户端骨架代码。这种方法在微服务架构中被广泛推崇，因为 API 契约是跨语言、跨团队协作的基石。
	-
- Kratos 项目的目录结构
  heading:: true
  collapsed:: true
	- 想写好 Kratos 项目，首先需要理解它的代码是如何组织的。
	- Kratos 的目录结构遵循官方推荐的标准化工程架构，清晰且便于维护。
	- `api`：
		- 用于存放 proto 文件的目录。在这里定义服务接口（`service`）和消息格式（`message`）。
		- `api` 层定义服务接口，由 `service` 层实现。
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
	- `internal`：存放项目主要业务逻辑的目录。
		- `internal` 目录是 Go 语言项目的一种约定，存放在此目录下的代码包只能被其父目录及子目录中的代码引用。这是一种强制的封装机制，确保项目的核心业务逻辑不会被外部项目随意导入，从而保证内部实现的独立性和可维护性。
		- `biz`：业务逻辑层（Business），存放核心业务逻辑（各个业务逻辑的组装）并被 service 层调用。在 biz 层定义 Repo 接口，规定数据操作的要求（接口），由 `data` 层实现。biz 层只关心调用方法，而不关心数据的来源。
		- `data`：数据访问层，存放操作数据库的代码。它实现了 `biz` 层定义的 `Repo` 接口，为 `biz` 层提供数据支持。
		- `service`：服务层，充当“胶水层”，将外部请求与内部业务逻辑连接起来。它实现 `.proto` 文件中定义的每一个 RPC 方法。接收 gRPC 或 HTTP 请求，进行参数处理后，调用 `biz` 层执行业务逻辑，将 `biz` 层返回的结果封装成响应消息返回。
		- `server`：服务器层，负责创建和配置 HTTP、gRPC 服务器，并将 `service` 层实现的服务实例注册到这些服务器上。
		- `conf`：定义配置结构体，用于加载和使用配置文件中的内容，如端口号、数据库地址等。
	- `third_party`：用于存放所有外部依赖的 `.proto` 文件。
	- **数据流向：**
		- `Request` → `server` → `service` → `biz` → `data` → `Database/Cache`
	- **Kratos 架构与 Spring Boot 架构的对比：**
		- `biz` 相当于 Spring Boot 的业务逻辑层 (`service`)，负责处理核心业务逻辑并编排业务流程。
		- `service` 相当于 Spring Boot 的控制器层 (`controller`)，接收并响应外部请求，进行参数校验，并调用下层业务。
		- `data` 相当于 Spring Boot 的数据访问层 (`dao`)，与数据源交互，负责数据持久化。
		- `service` 接收请求，调用 `biz` 处理业务，`biz` 层则通过 `data` 层的方法操作数据库。
- kratos 命令行工具
  heading:: true
  collapsed:: true
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
- **存根（Stub）：**
  collapsed:: true
	- 客户端存根和服务端骨架是一对孪生兄弟，都是由 API 定义（`.proto` 文件）自动生成的，它们共同构建了一个桥梁，让开发者可以忽略底层的网络细节，专注于业务逻辑的实现。
	- **客户端存根：**
		- 客户端存根是自动生成的一段代码，使得调用远程服务像调用本地函数一样简单。
		- 如果没有存根，你需要手动完成以下步骤：
			- 创建一个 HTTP/2 连接到服务器的 `IP:Port`。
			- 手动将请求参数（如字符串 `Kratos`）按照 Protobuf 的规则序列化成二进制字节流。
			- 将二进制字节流打包成 gRPC 请求，并通过 HTTP/2 连接发送。
			- 阻塞并等待服务器的响应。
			- 接收响应的二进制字节流。
			- 将二进制字节流按照 Protobuf 规则反序列化成可理解的响应数据。
			- 关闭连接。
		- 这个过程既繁琐又容易出错，且完全是重复性工作。
		- 使用存根后，你只需在客户端代码中像调用普通本地方法一样调用远程方法，存根会在幕后自动完成上述所有步骤（序列化、发送、接收、反序列化等）。因此，存根的核心作用是封装复杂的网络通信细节，为开发者提供一个简洁、清晰的本地调用接口。
		- 你可以将它视为一个“代理”或“中间人”。当你在客户端调用远程方法时，实际上是通过这个本地存根与远程服务进行交互，而不是直接与网络打交道。
	- **服务端存根：**
		- 在服务端的语境下，通常称其为“骨架（Skeleton）”，而非“存根”。它是由工具根据 API 定义（`.proto` 文件）自动生成的一段代码，通常表现为一个接口。
		- **服务端存根的工作流程（一次请求的旅程）：**
			- **接收网络数据**：Kratos 服务器底层的 gRPC 库监听指定端口，接收到一串二进制网络数据流。
			- **解码与反序列化**：框架识别出这是一个调用 `Greeter` 服务的 `SayHello` 方法的请求。它使用 Protobuf 规则，将二进制数据流反序列化为 Go 语言中对应的结构体 `*v1.SayHelloRequest`。
			- **调用“存根”**：框架完成了所有“繁琐”的工作，接下来它调用存根接口中对应的方法，并将反序列化后的 `*v1.SayHelloRequest` 对象作为参数传入。
			- **执行你的实现**：由于你的 `GreeterService` 实现了 `GreeterServer` 接口，框架会准确调用你在 `internal/service/greeter.go` 中编写的 `SayHello` 方法。
			- **专注业务**：在方法内部，你得到的已经是处理好的类型安全的 Go 结构体，无需关心网络、协议或序列化，只需专注于实现核心的业务逻辑。
			- **返回结果**：处理完请求后，你的方法返回一个 `*v1.SayHelloReply` 结构体和 `error`。
			- **序列化与发送**：框架接收到返回的 `*v1.SayHelloReply` 对象后，将其序列化成二进制数据流，并通过网络连接发送回客户端。如果返回了 `error`，框架会将其转换为标准的 gRPC 错误码。
- **UnimplementedXXXServer**
  collapsed:: true
	- `UnimplementedXXXServer` 是由 Go 的代码生成工具 `protoc-gen-go-grpc` 自动生成的默认实现占位结构体。当你在 `.proto` 文件中定义一个 `service XXX` 并执行代码生成命令时，该工具会为该服务自动生成对应的 `UnimplementedXXXServer`。
	- **提供默认的 “未实现” 行为：**
	  collapsed:: true
		- 它为 `.proto` 中定义的所有 gRPC 方法提供了默认实现，这些方法会返回一个“未实现”的错误，用于提示该功能尚未被实现。
		- 通过将 `UnimplementedXXXServer` 嵌入到 `XXXService` 结构体中，`XXXService` 就能自动继承它的方法，从而获得基础的接口实现，避免手写占位实现。
		- **示例：**
			- ```go
			  func (UnimplementedContainerLogServiceServer) GetContainerLogs(context.Context, *GetContainerLogsRequest) (*GetContainerLogsReply, error) {
			  	return nil, status.Errorf(codes.Unimplemented, "method GetContainerLogs not implemented")
			  }
			  ```
	- **为什么是“值嵌入”而不是“指针嵌入”？**
	  collapsed:: true
		- 之所以要求“值嵌入”而不是“指针嵌入”，是为了避免在运行时调用默认实现（那些返回 `codes.Unimplemented` 的方法）时因为底层指针是 `nil` 而触发 panic。
		- 在生成代码里，UnimplementedXXXServer 默认方法都是值接收者实现。
		- 如果你把 `UnimplementedContainerLogServiceServer` 作为指针字段嵌入（例如 `*v1.Unimplemented...`），结构体的零值字段就是 `nil`。
		- 当 gRPC 框架去调用默认方法时，Go 需要对嵌入字段调用值接收者方法，可这些方法是以值接收者定义的，因为方法是值接收者，Go 会尝试“自动解引用”指针接收者，让 `*ptr` 来调用，但字段是 `nil`，Go 在调用时会对 `nil` 指针做隐式解引用，最终导致 panic。（相当于显式写成 `(*ptr).GetContainerLogs` 时直接解引用了一个 `nil`）。
		- **为避免这种隐蔽的崩溃，注册逻辑里专门做了检查：**
			- 生成代码在注册服务时会做一次检查，注册时通过接口断言把服务对象转换成一个包含 `testEmbeddedByValue()` 的接口并立即调用。
			- ```go
			  func RegisterContainerLogServiceServer(s grpc.ServiceRegistrar, srv ContainerLogServiceServer) {
			  	// If the following call pancis, it indicates UnimplementedContainerLogServiceServer was
			  	// embedded by pointer and is nil.  This will cause panics if an
			  	// unimplemented method is ever invoked, so we test this at initialization
			  	// time to prevent it from happening at runtime later due to I/O.
			  	if t, ok := srv.(interface{ testEmbeddedByValue() }); ok {
			  		t.testEmbeddedByValue()
			  	}
			  	s.RegisterService(&ContainerLogService_ServiceDesc, srv)
			  }
			  ```
			- 只有值嵌入时，这个方法才会存在且接收者不是 `nil`：你嵌入的匿名字段会在零值时自动构造出一个“空结构体值”，调用是安全的。
			- 如果开发者用指针嵌入，断言虽然能通过，但字段是 `nil`，调用 `testEmbeddedByValue()` 时会因为底层指针为 `nil` 而 panic，从而在启动阶段就暴露问题，从而避免在实际 RPC 调用时因为 nil 指针而产生更难排查的崩溃。
			- 利用了 Go 接口的动态调用：把服务对象转成接口并立即调用方法；若是值嵌入，调用安全通过；若是指针嵌入且为 `nil`，则在注册时立刻 panic，督促开发者改用值嵌入。这确保默认未实现逻辑在运行时总是有一个有效的接收者实例，避免潜在的 `nil` 解引用崩溃。
	- **获得自动的前向兼容**：
	  collapsed:: true
		- 当 `.proto` 文件中新增 RPC 方法时，重新生成的 `UnimplementedXXXServer` 会自动包含这些方法的默认实现（返回 `codes.Unimplemented`）。
		- 只要你的服务结构体采用“值嵌入”方式，这些默认方法就会自动提升到服务结构体中，即使你还没来得及实现新方法，也能编过、跑起来且有可预期的返回。
		- 这样一来，即使接口更新，也无需立刻修改代码就能继续通过编译，避免旧服务在接口变更时无法编译的问题。
		- 唯一的缺点是，新增加的 RPC 方法会暂时返回“未实现”的错误，但服务仍可正常编译和部署。
	- **mustEmbedUnimplementedContainerLogServiceServer 有什么用？**
	  collapsed:: true
		- **核心作用**：其唯一目的是劝导你嵌入 `UnimplementedContainerLogServiceServer` 结构体。
		- 它本身是个空方法，不会被调用。
		- 根据 Go 语言的规则，任何想实现接口的结构体都必须提供这个方法。它就像一个密码：
			- 接口 `ContainerLogServiceServer` 说：“想实现我，就必须提供这个密码（方法）”。
			- 编译器在检查你的代码时，会看你有没有提供这个“密码”。没有就报错。
			- 而 `UnimplementedContainerLogServiceServer` 这个结构体天生就带着这个“密码”。所以你只要把它嵌入你的结构体，编译器就满意了。
		- 通过增加“必须实现一个奇怪的空方法”这个小麻烦，并用方法名直接提示你应该怎么做，来引导你走上最简单且最正确的道路——即通过嵌入 `Unimplemented...` 结构体来自动满足这个要求。
- **Kratos 的开发流程：**
  collapsed:: true
	- **定义 API**：使用 Protobuf 定义服务的功能、输入数据和返回数据。
	- **编写服务层（Service）代码**：
		- 创建 Service 并实现 Protobuf 中定义的 API 接口（通过工具自动生成）。
	- **编写业务层（Biz）代码**：
		- 定义业务模型（DO，Domain Object），业务模型专注于业务逻辑，而非数据库表结构。
		- 定义仓库接口（Repo），抽象数据操作，不关心具体实现。
		- 编写业务用例，结合业务模型、仓库接口和业务逻辑，完成具体的业务流程。
		- 让业务层被服务层调用。
	- **编写数据层（Data）代码**：
		- 实现业务层定义的仓库接口，编写数据库客户端连接及操作数据库的代码。
		- 定义与数据库表结构对应的持久化对象（PO，Persistent Object）。
	- 创建数据库表并进行迁移，确保数据模型与数据库一致。
	- **依赖注入**：将各层的构造函数注册到各自的 `ProviderSet` 中。
	- **服务器注册**：将 Service 注册到 Kratos 的路由系统中，确保服务可以处理外部请求。
	- 执行 `wire` 命令，生成依赖注入代码。
	- 采用“由内向外、自底向上（Bottom-Up）”的顺序，确保在编写每一层代码时，其依赖的底层组件已存在并可用。这样，你始终站在坚实的基础上，而不是在“空中楼阁”上写代码。就像盖房子，我们首先打好坚实的地基（Model, pkg），然后搭建承重墙（Data），接着构建核心房间（Biz），最后装修门面（Service）。
- **Usecase 是什么：**
  collapsed:: true
	- Usecase（用例）表示某个特定的业务操作或任务。
	- Usecase 层负责处理用户请求的具体业务逻辑，通常不涉及细节问题，而是专注于协调系统各个部分完成业务任务。
	- Usecase 是核心业务逻辑层。当 Controller 层收到用户的注册请求后，会调用 `RegisterUserUsecase`。`RegisterUserUsecase` 负责处理用户注册的业务逻辑（如验证输入、保存数据），并通过 `UserRepo` 保存用户信息。
	- 用例本身不负责操作的具体实现（例如如何查询商品、如何调用支付接口），它的任务是将这些操作组合起来，协调各个步骤的执行，确保整个流程顺畅。它像一个“指挥家”，作为业务流程的编排者，负责协调和组织操作，但并不实现具体操作。
- **`cleanup` 函数：**
  collapsed:: true
	- `cleanup` 是一个专门用于清理资源的函数，在应用程序关闭时，它负责释放资源、关闭数据库连接、断开缓存客户端等，以防止资源泄漏。
	- 将 `cleanup` 定义在函数内部并返回，而不是直接在外部定义，有以下几个原因：
		- 使其能够直接访问函数内部的一些上下文信息，因为那些需要清理的资源通常是在函数内部初始化的。
		- 允许外部控制执行时机并支持延迟执行：这样可以让调用者决定何时执行 `cleanup`，而不是在程序运行时立即执行，从而确保在合适的时机（例如应用退出时）释放资源。
- **为什么要分层设计模型？**
  collapsed:: true
	- 不分层设计的弊端在于，所有层共用同一个模型，导致层与层之间耦合过高。一旦模型发生变动，所有层都需进行修改。此外，这种设计容易引发隐私泄露等风险，比如直接返回数据库层的模型，其中许多隐私字段未被妥善处理。
	- **不分层设计：**
		- ```go
		  // 不分层设计：所有层共用同一个模型
		  package model
		  
		  type User struct {
		      ID           int64  `json:"id"`
		      Username     string `json:"username"`
		      PasswordHash string `json:"-"` // 忽略密码
		      LastLoginIP  string `json:"-"` // 隐私字段，误返回 API
		  }
		  
		  // biz 层
		  func GetUser(id int64) (*model.User, error) {
		      return repo.GetByID(id) // 直接返回包含 LastLoginIP 的 User 模型
		  }
		  
		  // service 层
		  func GetUser(ctx context.Context, req *pb.GetUserRequest) (*model.User, error) {
		      return biz.GetUser(req.GetId()) // 直接返回 model.User，可能泄露 LastLoginIP
		  }
		  ```
	- **分层设计：**
		- ```go
		  // data 层模型（数据库存储）
		  package data
		  
		  type User struct {
		      ID           int64  `gorm:"primarykey"`
		      Username     string `gorm:"unique"`
		      PasswordHash string
		      LastLoginIP  string // 数据库保存该字段
		  }
		  
		  // biz 层模型（业务逻辑）
		  package biz
		  
		  type User struct {
		      ID       int64
		      Username string
		      // 没有 LastLoginIP 字段
		  }
		  
		  // 转换逻辑：将 data.User 转换为 biz.User
		  func convertToBiz(po *data.User) *biz.User {
		      return &biz.User{
		          ID:       po.ID,
		          Username: po.Username,
		          // LastLoginIP 被“过滤”掉
		      }
		  }
		  
		  // service 层
		  func GetUser(ctx context.Context, req *pb.GetUserRequest) (*biz.User, error) {
		      userData, err := repo.GetByID(req.GetId())
		      if err != nil {
		          return nil, err
		      }
		      return convertToBiz(userData), nil // 返回简化后的 biz.User
		  }
		  ```
- [[Kratos 的日志]]
- **Kratos 各层参数校验的最佳实践**
  collapsed:: true
	- 在 Kratos 项目中，每个层级的校验负责不同类型的参数。
	- **服务层（Service Layer）**
		- 服务层主要进行初步的、简单的校验，确保传入的数据符合基本要求，避免无效数据进入业务层，不涉及复杂逻辑或数据库查询。
		- **语法校验：**
			- 检查字符串是否为空。
			- 检查整数是否在合理范围内。
			- 检查邮箱格式是否正确。
			- ```proto
			  message AddRegistryRequest {
			      string name = 1 [(validate.rules).string.min_len = 1];  // 检查 name 不为空
			      string email = 2 [(validate.rules).string.email = true];  // 检查 email 格式
			      int32 age = 3 [(validate.rules).int32.gte = 18];  // 检查 age 大于或等于 18
			  }
			  ```
		- **基本语义校验：**
			- 如果提供了用户名，密码字段也必须填写。
			- ```proto
			  if req.Username != "" && req.Password == "" {
			      return errors.New("password is required when username is provided")
			  }
			  ```
	- **业务层（Biz Layer）**
		- 业务层负责进行复杂的业务规则校验，确保操作合法并符合系统的核心规则和数据一致性要求。这些校验通常涉及数据库查询或外部服务调用，以确保业务逻辑的正确性，不会破坏系统的基本规则。
		- **复杂业务规则校验：**
			- 检查用户名是否已存在。
			- 验证仓库地址是否可达。
			- ```go
			  // 检查仓库名称是否已存在
			  if repositoryExists(req.Name) {
			      return errors.New("repository name already exists")
			  }
			  
			  // 验证仓库地址是否可达
			  if !isReachable(req.Address) {
			      return errors.New("repository address is unreachable")
			  }
			  ```
		- **业务不变量校验：**
			- 确保用户余额足够进行转账。
			- ```go
			  // 确保用户余额足够
			  if userBalance < req.Amount {
			      return errors.New("insufficient balance")
			  }
			  ```
	- **数据层（Data Layer）**
		- 数据层依赖数据库的约束条件来确保数据符合数据库的规则，避免不一致或错误数据进入系统。
		- 参数校验应该在 biz 层进行。biz 层负责业务逻辑和参数校验，而 data 层主要负责数据访问逻辑。
- **为啥 Biz 层能直接用 Model？这耦合能接受吗？**
  collapsed:: true
	- **问题背景：**
		- 写代码的时候，我发现在 `biz` 层的代码里，直接就 `&model.Registry{...}` 这样去创建一个 `model` 结构体了。心里犯嘀咕：Kratos 这种分层架构，不是讲究解耦吗？这么直接引用，算不算一种不好的“耦合”？
	- **Biz 层与 Model 层的耦合：一种必要的设计**
		- **`Model` 层：数据的“模具”**
			- 它只做一件事：定义核心数据应该长什么样子。例如，一个 `UserModel` 会规定“用户”必须包含 `ID`、`Name` 等字段。它为业务操作提供了一个标准化的数据结构蓝本。
		- **`Biz` 层：业务的“加工者”**
			- 它负责执行具体的业务逻辑。例如，在处理“用户注册”时，`Biz` 层必须使用 `UserModel` 这个“模具”，将接收到的数据进行校验、组装，最终“浇筑”出一个结构完整、数据合规的用户实体，以备持久化。
		- `Biz` 和 `Model` 是一对相辅相成的伙伴。一个负责定义数据实体（出实体），一个负责实现业务逻辑（出主意）。如果 `Biz` 层不认识 `Model` 这个“模具”，它就无法创建出任何有效的数据交给 `Data` 层处理，整个业务流程便无从谈起。
	- **解耦解的是谁？**
		- **解耦 API 层和 Biz 层：**
			- `biz` 层不直接使用 `api` 层定义的请求结构体（如 `v1.AddRegistryRequest`），而是定义了自己的“工作参数”（`biz.AddRegistryParams`）。
			- `api` 层作为“前台接待”，负责与外部交互，接收到的“客户订单”（API 请求）格式可能会经常变化。为了避免前台的变化影响到“后厨”（`biz` 层），我们规定，前台必须将客户订单转化为后厨能够理解的、标准化的“配菜单”（`biz` 参数），再交给后厨。这样，即使客户点菜的方式发生变化，后厨的工作流程依然不受影响。
		- **解耦 Biz 层和 Data 层：**
			- `biz` 层不直接调用 `data` 层的具体实现，而是通过调用 `data` 层提供的**接口**（如 `biz.RegistryRepo`）。
			- 后厨（`biz` 层）完成一道菜（构建好 `model` 数据）后，需要将其存入仓库（数据库）。后厨不会直接去 MySQL 仓库，而是按下墙上的按钮（调用接口），喊道：“仓库管理员，来取货！”至于值班的管理员是 MySQL 还是 PostgreSQL，后厨并不关心，只要能把货存好就行。这样，哪怕我们把 MySQL 换成 PostgreSQL，后厨的工作流程也不会受影响。
			  id:: 68947f29-67a0-4486-a6b7-fac1172ba3d1
- 在 service 把 biz 和 api 解耦：proto 的 req 和 resp 不要直接传入 biz，而是转换成 dto 再传，虽然看起来做多了一步复制对象的操作，但是实现了biz 和 api 层的解耦，甚至是 data 和 api 层的解耦，之后修改 api 层的代码，比如修改包名，就不需要改 biz 层的代码，否则要改biz层的函数签名。
-