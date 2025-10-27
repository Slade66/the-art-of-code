collapsed:: true

**基于字符串的错误模型在分布式系统中的不足**

	- 在分布式系统中，一个简单的错误字符串（如 `"record not found"`）虽然便于开发者理解，但对系统中的自动化组件（如 API 网关、服务网格、监控系统）几乎无用。因为这些组件需要可被机器解析的结构化信息，才能据此执行自动化策略。
	- **在微服务架构中，这种局限性带来了多方面的挑战：**
		- **自动化治理决策困难**：API 网关如何判断一个下游服务的失败是由于客户端请求错误（不应重试）还是临时的网络抖动（应该重试）？如果错误仅仅是一个字符串，网关将无法做出智能决策。
		- **跨语言通信障碍**：微服务生态系统通常是多语言的。一个由 Go 服务产生的错误字符串，对于一个 Java 或 Python 服务来说，可能难以统一解析和处理，导致错误契约的脆弱性。
		- **监控与告警的粒度问题**：如何对特定类型的业务错误（例如，“库存不足”）进行精确的监控和告警，而不是笼统地监控所有 500 内部服务器错误？基于字符串的错误聚合和分类极其困难且不可靠。
- **Kratos 的错误：`errors.Error` 结构体**
  collapsed:: true
	- 一个 Kratos 的错误值示例：
		- ```json
		  {
		    "code": 404,
		    "reason": "USER_NOT_FOUND",
		    "message": "user 'foo' not found",
		    "metadata": {
		      "user_id": "foo"
		    }
		  }
		  ```
	- 这个结构清晰地展示了 Kratos 错误模型的四个核心组件：`code`、`reason`、`message` 和 `metadata`。
		- **Code**：整数类型，对应传输层的标准状态码（如 HTTP 404、500），用于标识错误的大类。它主要服务于机器和基础设施组件（如 API 网关、负载均衡器等），帮助它们根据统一规则执行重试、熔断等通用逻辑。Kratos 会确保该字段在 HTTP 与 gRPC 等协议间保持语义一致。
		- **Reason**：字符串类型，表示业务层面的具体错误标识（如 `USER_NOT_FOUND`、`ORDER_EXPIRED`）。它比 Code 更精细，主要供客户端和开发者使用，可用于在代码中根据不同业务错误编写差异化的处理逻辑。
		- **Message**：人类可读的错误描述（如 `"user 'foo' not found"`），用于在日志或界面上展示，帮助开发者或终端用户理解错误原因。由于内容可能随文案或上下文变化，不建议在程序逻辑中依赖它。
		- **Metadata**：键值对集合（`map[string]string`），用于携带丰富的上下文信息，如请求 ID、追踪 ID、资源标识符、参数校验字段等。这些信息有助于日志分析、链路追踪和故障定位，是可观测性体系的重要补充。
		- 一个 Kratos 风格的错误必须包含 code、reason、message。
	- 这些字段旨在提供现代可观测性所需的关键信息，用于明确失败的类型（如瞬时性、永久性、客户端或服务端错误）、业务层的唯一标识符，以及错误发生时的系统上下文（例如关联的用户或资源 ID）。
	- 一个 Kratos 错误对象，本质上就是一个预先打包好的、针对失败事件的结构化日志条目，使其能够被现代可观测性平台原生消费。
	- **实现 `Unwrap` 接口**：`*errors.Error` 类型实现了标准的 `Unwrap() error` 方法 。这意味着你可以将一个底层错误（cause）包装进一个 Kratos 错误中，并且之后仍然可以使用标准的 `errors.Unwrap`、`errors.Is` 和 `errors.As` 函数来访问和检查这个被包装的错误。
	- **添加底层原因**：通过 `WithCause(cause error)` 方法，可以为一个 Kratos 错误附加一个底层的错误原因 。这在将一个来自第三方库或标准库的错误（例如，数据库驱动返回的 `sql.ErrNoRows`）转换为一个具有业务含义的 Kratos 错误时非常关键。
		- ```go
		  import "database/sql"
		  
		  //... 在数据访问层...
		  dbErr := sql.ErrNoRows // 模拟数据库返回错误
		  
		  if errors.Is(dbErr, sql.ErrNoRows) {
		      // 将底层数据库错误包装成一个业务错误
		      // 同时保留原始错误作为 cause
		      kratosErr := errors.NotFound("USER_NOT_FOUND", "user not found").WithCause(dbErr)
		      return kratosErr
		  }
		  ```
		- 上层调用者既可以使用 `errors.Is(kratosErr, errors.NotFound("USER_NOT_FOUND", ""))` 来检查业务错误，也可以使用 `errors.Is(kratosErr, sql.ErrNoRows)` 来检查底层的数据库错误，实现了信息的完整保留。
	- **使用 `WithMetadata` 丰富错误以增强可调试性**
		- 通过这种方式，当这个错误最终被日志系统记录下来时，它将包含所有必要的上下文信息，使得开发者能够迅速定位问题，而无需在代码的多个地方手动记录这些信息。
		- ```go
		  func GetUser(ctx context.Context, userID string) (*User, error) {
		      user, err := repo.FindUser(ctx, userID)
		      if err!= nil {
		          // 假设 repo.FindUser 返回一个 Kratos 错误
		          // 在返回前，添加请求相关的元数据
		          return nil, errors.FromError(err).WithMetadata(map[string]string{
		              "request_user_id": userID,
		              "trace_id": tracing.TraceID(ctx),
		          })
		      }
		      return user, nil
		  }
		  ```
- **Kratos 创建错误：**
  collapsed:: true
	- **基础构造函数**：`errors.New(code int, reason, message string)`
		- 它接收一个状态码、一个业务原因和一个用户消息，并返回一个 `*errors.Error` 实例。
		- ```go
		  // 创建一个表示资源未找到的错误
		  err := errors.New(404, "USER_NOT_FOUND", "user with id 123 not found")
		  ```
	- **格式化构造函数**：`errors.Newf(code int, reason, format string, a ...any)`
		- 功能与 `errors.New` 类似，但允许像 `fmt.Sprintf` 一样格式化 `message` 字符串。
		- ```go
		  userID := 123
		  err := errors.Newf(404, "USER_NOT_FOUND", "user with id %d not found", userID)
		  ```
	- **预定义辅助函数**：
		- 为了提升代码可读性和开发效率，Kratos 的 `errors` 包预定义了一组与标准 HTTP 状态码对应的辅助函数，如 `errors.BadRequest()`、`errors.NotFound()`、`errors.InternalServer()` 等。
		- 这些函数本质上是 `errors.New` 的快捷封装，内部已预置相应的 `code` 值（如 400、404、500 等），开发者只需传入 `reason` 和 `message` 即可快速创建结构化错误对象。
		- 同时，包内还提供了对应的判断函数（如 `IsBadRequest()`、`IsNotFound()`），便于在错误处理逻辑中进行类型识别和分支控制。
- **Kratos 检查错误：**
  collapsed:: true
	- 当从其他函数接收到一个 `error` 类型的返回值时，需要有方法来检查它是否是一个特定的 Kratos 错误。
	- **`errors.FromError(err)`**
		- Kratos 提供的工具函数，用于将任意 `error` 转换为 Kratos 定义的结构化错误，便于统一提取 `Code`、`Reason`、`Message` 等字段。
		- **其行为如下：**
			- 若 `err` 为 `nil`，则返回 `nil`。
			- 若 `err` 已是 Kratos 错误，原样返回。
			- 若为普通 `error`，则包装为新的 Kratos 错误，默认 `code=500`、`reason=""`、`message=err.Error()`。
		- **示例：**
			- ```go
			  if e := errors.FromError(err); e != nil {
			      if e.Reason == "USER_NOT_FOUND" && e.Code == 404 {
			          // 针对用户未找到的特定处理逻辑
			          log.Printf("User not found, metadata: %v", e.Metadata)
			      }
			  }
			  ```
	- **`errors.Is(err, target error)`**
		- 可以先创建一个仅包含 `code` 和 `reason` 的临时“目标错误”，然后使用 `errors.Is` 判断 `err` 是否与其匹配。`errors.Is` 会自动遍历整个错误链，完成匹配检查。
		- ```go
		  // 创建一个目标错误用于比较
		  target := errors.NotFound("USER_NOT_FOUND", "")
		  
		  if errors.Is(err, target) {
		      // 确认这是一个用户未找到的错误
		      // 这里的 err 可能是被包装过的，但 Is 依然能正确识别
		  }
		  ```
- **核心原则：错误应定义在协议文件（ `.proto` ）中，而非随处  `errors.New("xxx")`**
  collapsed:: true
	- **传统开发模式的弊端：**在传统的开发模式中，错误码和错误信息通常以常量或枚举的形式硬编码在服务端的代码中。客户端或其他服务需要通过查阅文档或直接复制代码来了解这些错误定义，这种方式非常脆弱且容易出错。
	- **拥抱 API 优先的错误管理方法：**
		- Kratos 框架中最具特色和威力的功能之一，便是其倡导的“API 优先”的错误管理方法。该方法将业务错误的定义提升到了与 API 请求和响应同等重要的地位，这种方法从根本上解决了微服务架构中错误契约管理混乱的难题。
		- Kratos 认为，微服务之间传递的错误本质上也是一种 API 响应，因此应像正常响应一样，具备清晰、预先定义的结构。换言之，API 不仅要定义成功时的数据格式，也应明确失败时的返回内容。
		- Kratos 将错误定义视为服务对外契约不可分割的一部分 。既然服务的接口（Service）、消息（Message）都是通过 `.proto` 文件来定义的，那么服务可能返回的业务错误（Error）也理应在同一个地方进行定义。
		- Kratos 建议将错误定义集中在统一的协议文件（如 `.proto`）中，并通过代码生成器为各语言自动生成错误常量和构造函数。这样，无论系统包含多少服务、使用何种语言（Go、Java 等），都能共享统一的错误码与定义，实现跨服务、跨语言的一致性，降低维护成本，避免因定义不一致造成的调试困难。
- **在 `.proto` 文件中定义 `ErrorReason`**
  collapsed:: true
	- 在 Kratos 中，定义业务错误的核心是创建一个名为 `ErrorReason` 的 Protobuf `enum`（枚举）类型。
	- **代码示例：**
		- ```protobuf
		  syntax = "proto3";
		  
		  package api.blog.v1;
		  
		  import "errors/errors.proto";
		  
		  option go_package = "your_project/api/blog/v1;v1";
		  
		  enum ErrorReason {
		    // 为所有错误设置一个默认的 HTTP 状态码为 500 (Internal Server Error)
		    option (errors.default_code) = 500;
		  
		    // 为 USER_NOT_FOUND 单独指定 HTTP 状态码为 404 (Not Found)
		    USER_NOT_FOUND = 0 [(errors.code) = 404];
		  
		    // 为 CONTENT_MISSING 单独指定 HTTP 状态码为 400 (Bad Request)
		    CONTENT_MISSING = 1 [(errors.code) = 400];
		  }
		  ```
		- `option (errors.default_code) = 500;`：这个选项设置在 `enum` 的顶层，为该枚举中定义的所有错误原因指定一个默认的 HTTP 状态码。如果某个错误原因没有单独指定 `code`，它将使用这个默认值。
		- `[(errors.code) = 404]`：这个选项附加在单个枚举值的后面，用于覆盖默认的 `code`。它为特定的 `reason` 指定了其对应的 HTTP 状态码。
	- **使用 `protoc-gen-go-errors` 生成代码**
		- 定义好包含 `ErrorReason` 的 `.proto` 文件后，可使用 `protoc` 编译器结合 Kratos 提供的专用插件 `protoc-gen-go-errors` 生成对应的 Go 代码。
		- 代码生成后，会在指定的输出目录下创建一个 `xxx_errors.pb.go` 文件。这个文件包含了以下类型的辅助函数：
			- **错误检查函数**：为每个错误原因生成一个 `IsXXX` 函数。
			- **错误创建函数**：为每个错误原因生成一个 `ErrorXXX` 函数，用于创建对应的 `*errors.Error` 实例。
		- 在业务逻辑代码中，可以这样使用这些生成的函数：
			- ```go
			  package biz
			  
			  import "your_project/api/blog/v1"
			  
			  func (uc *BlogUsecase) GetArticle(ctx context.Context, id int64) (*Article, error) {
			      //...
			      if articleNotFound {
			          // 使用生成的创建函数来返回一个标准的 Kratos 错误
			          // "article with id %d not found" 将成为 message
			          // reason 会是 "USER_NOT_FOUND"
			          // code 会是 404 (根据.proto 定义)
			          return nil, v1.ErrorUserNotFound("article with id %d not found", id)
			      }
			      //...
			  }
			  
			  // 在上层调用代码中
			  article, err := usecase.GetArticle(ctx, 1)
			  if err!= nil {
			      // 使用生成的检查函数，代码清晰、类型安全且不会因为字符串拼写错误而出错
			      if v1.IsUserNotFound(err) {
			          // Handle user not found case
			      }
			  }
			  ```
- **分层架构中错误处理的最佳实践**
  collapsed:: true
	- 在靠近错误源头的底层，保持错误的原始性；在中间的业务层，进行错误的转换和上下文添加；在最顶层的应用边界，进行错误的最终处理（如日志记录和响应生成）。
	- **数据层 ( `data` )：传递原始驱动错误**
		- 数据层负责与外部数据源（如数据库、缓存、第三方 API）进行交互。在 Kratos 的标准项目布局中，这一层对应于 `data` 包 。
		- 这一层的核心原则是**直接返回**由底层驱动或客户端库产生的原始错误，**不要进行任何包装或转换** 。例如，如果使用 GORM 操作数据库，当查询不到记录时，就应该直接返回 `gorm.ErrRecordNotFound`。
		- 数据层应该保持业务无关性。它的任务是忠实地报告数据操作的结果。在这一层进行错误包装会过早地引入业务上下文，使得这个数据访问组件难以在其他业务场景中复用。保持错误的原始性，将解释错误的权力交给了上层的调用者。
		- 在 `data` 层，将错误原封不动地抛出，并在顶层的 Middleware/Interceptor 中统一处理日志记录。
	- **业务逻辑层 ( `biz` )：关键的转换点**
		- 业务逻辑层是应用核心业务规则的所在地，对应于 Kratos 布局中的 `biz` 包 。
		- 它应该捕获来自 `data` 层的、与实现细节相关的特定错误（如 `gorm.ErrRecordNotFound`），并将其**转换并包装**成一个对上层有意义的、领域特定的 Kratos 业务错误（如 `api.ErrorUserNotFound(...)`）。在转换时，应使用 `WithCause` 方法保留原始的底层错误，以便于深度调试。
		- 经过 `biz` 层的处理后，应用的其他部分（如 `service` 层和 API handler）只需关心定义在 `.proto` 文件中的、标准化的业务错误，而完全无需了解底层使用的是 GORM、`database/sql` 还是其他任何数据存储技术。
	- **服务层 ( `service` )：协调与传递**
		- 服务层负责编排对 `biz` 层的调用，处理数据传输对象（DTO）与领域对象（DO）之间的转换，并为 API 接口准备数据。
		- 通常情况下，`service` 层只需将从 `biz` 层接收到的 Kratos 错误**直接向上传递**。在某些复杂的业务流程中，`service` 层也可以在传递错误之前，使用 `WithMetadata` 为其添加更多与当前请求或业务流程相关的上下文信息。
		- `service` 层的主要职责是流程编排，而不是定义新的业务错误。它作为 `biz` 层和传输层之间的桥梁，确保错误信息能够被完整地传递。
	- **传输边界 (API Handlers)：最终处理与响应**
		- 这一层是应用的边界，负责处理传入的 HTTP 请求或 gRPC 调用，调用 `service` 层，并最终生成响应。
		- 这是错误流程中**唯一应该“处理”错误**的地方。API handler 在收到 `service` 层返回的 `error` 后，应该做的仅仅是**将这个 `error` 直接 `return`**。框架的 `ErrorEncoder`（对于 HTTP）或 gRPC 服务器的内置机制会接管后续的一切，包括日志记录（通常在中间件或拦截器中完成）和生成最终的协议响应。
		- 一个错误应该只被记录一次，而记录的最佳位置就在应用的入口/出口处。如果在底层或业务层记录日志，当一个错误经过多个函数调用栈时，会导致日志的重复和混乱，给问题排查带来巨大干扰。
	- **示例：**
		- **HTTP Handler (传输层)**：接收到 `GET /users/123` 请求，解析出 `userID` 为 "123"，然后调用 `service.GetUser(ctx, 123)`。
		  logseq.order-list-type:: number
		- **Service Layer (`service`)**：调用 `biz.GetUser(ctx, 123)`。
		  logseq.order-list-type:: number
		- **Biz Layer (`biz`)**：调用 `data.FindUserByID(ctx, 123)`。
		  logseq.order-list-type:: number
		- **Data Layer (`data`)**：执行数据库查询，例如 `db.First(&user, 123)`。GORM 未找到记录，返回 `gorm.ErrRecordNotFound`。`data` 层的方法直接 `return nil, gorm.ErrRecordNotFound`。
		  logseq.order-list-type:: number
		- **Biz Layer (`biz`)**：接收到 `gorm.ErrRecordNotFound`。代码执行 `if errors.Is(err, gorm.ErrRecordNotFound)` 判断，条件成立。然后，它创建一个业务错误并返回：`return nil, v1.ErrorUserNotFound("user with id %d not found", 123).WithCause(err)`。
		  logseq.order-list-type:: number
		- **Service Layer & Handler (传输层)**：`service` 层接收到这个 Kratos 错误，并直接将其返回。HTTP handler 也同样直接 `return err`。
		  logseq.order-list-type:: number
		- **Middleware/Interceptor (传输层)**：位于 handler 之前的日志中间件捕获到这个返回的错误。它调用 `errors.FromError(err)`，提取出 `reason`、`message`、`metadata` 和 `cause`，并将这些结构化信息记录到日志中。
		  logseq.order-list-type:: number
		- **`ErrorEncoder` (传输层)**：框架的 `ErrorEncoder` 最后被调用。它接收到这个 Kratos 错误，从中读取 `Code` (404)，并设置 HTTP 响应状态码为 404。然后，它将整个 `*errors.Error` 对象序列化为 JSON，写入 HTTP 响应体，发送给客户端。
		  logseq.order-list-type:: number
-