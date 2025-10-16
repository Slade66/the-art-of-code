- **为什么需要日志？**
	- 日志是可观测性（Observability）中非常重要的一环，掌握好它能让你在开发和排查问题时事半功倍。
	- 清晰规范的日志是微服务稳定运行的基础，而混乱的日志不仅难以定位问题，还会产生大量无效信息，浪费存储和计算资源。
- **Kratos 的日志分层设计：**
	- 当你记录一条日志时，最关注的是“记录什么内容”（What）以及“使用什么级别”（Level），比如记录一条 Info 级别的日志：“用户信息获取成功”。
	- 而从整个系统的角度，还需要考虑日志“如何输出”（How）以及“输出到哪里”（Where）。例如：
		- 在开发环境中，希望以彩色文本格式输出到控制台，便于阅读和调试；
		- 在测试环境中，希望以 JSON 格式输出到文件，方便后续分析和归档；
		- 在生产环境中，希望将日志发送到远程日志系统，如阿里云日志服务或 Fluentd，便于集中管理和监控。
	- **`Helper`（高层接口）**：这是你在**业务代码中应直接使用**的接口，专注于“记录什么”。它提供了如 `Info()`、`Errorf()`、`Warnw()` 等简单易用的方法，你只需调用对应方法并传入日志内容与级别即可。
	- **`Logger`（底层接口）**：用于**对接具体的日志实现**，专注于日志的“输出方式”和“输出位置”。你可以使用 Kratos 提供的实现，也可以自定义一个 `Logger`，来决定日志的格式、写入位置或发送目标（如文件、本地控制台、远程日志服务等）。
	- 业务代码只与 `Helper` 交互，`Helper` 会将日志请求标准化后，交由背后可随时替换的 `Logger` 执行。
	- **这样做的好处是什么？**
		- **灵活性与可扩展性**：
			- 当技术选型发生变更（例如公司从 `Zap` 迁移到自研日志平台），你**无需改动任何业务代码**，只需提供一个新的 `Logger` 实现，并在项目入口（如 `main.go`）中替换旧的 `Logger` 即可，整个过程对业务层完全透明。
			- 业务代码与底层日志实现解耦，使得你可以在不同环境中自由切换日志输出方式（如从标准输出切换到 Zap 或 Fluentd），而**无需修改业务逻辑**。
		- **关注点分离**：
			- **业务开发者（`Helper` 的使用者）**：只需关注“记录哪些业务信息”，不必关心日志格式或输出方式，代码更简洁、逻辑更清晰。
			- **基础设施开发者（`Logger` 的实现者）**：专注于日志的处理和输出，不必涉及具体业务逻辑。
- **`Logger` 接口：**
	- **源代码：**
		- ```go
		  // Logger is a logger interface.
		  type Logger interface {
		  	Log(level Level, keyvals ...any) error
		  }
		  ```
	- `Logger` 接口设计非常精简，仅包含一个方法 `Log`，用于记录一条日志事件。该方法接收两个参数：
		- `level Level`：枚举类型，表示日志级别，如 `Debug`、`Info`、`Warn`、`Error`、`Fatal`，用于标识日志的严重程度。
		- `keyvals`：一个键值对序列，接受任意数量的参数（必须是偶数个，成对出现）。例如：`"msg", "hello", "user_id", 123`。其中 `key` 通常为字符串，`value` 可为任意类型。
	- `Logger` 接口只定义了一个日志事件（Log Event）的最小契约：**一条日志 = 一个等级 + 一组键值对**。
	- Kratos 在 `contrib/log` 目录下已内置了多种 `Logger` 实现，方便对接常见日志系统：
		- `zap`：集成 Uber 开源的高性能日志库 Zap。
		- `fluent`：对接 Fluentd 日志收集器。
		- `aliyun`：对接阿里云日志服务。
- **`Helper` 结构体：**
	- **源代码：**
		- ```go
		  // Helper is a logger helper.
		  type Helper struct {
		  	logger  Logger
		  	msgKey  string
		  	sprint  func(...any) string
		  	sprintf func(format string, a ...any) string
		  }
		  
		  // NewHelper new a logger helper.
		  func NewHelper(logger Logger, opts ...Option) *Helper {
		  	options := &Helper{
		  		msgKey:  DefaultMessageKey, // default message key
		  		logger:  logger,
		  		sprint:  fmt.Sprint,
		  		sprintf: fmt.Sprintf,
		  	}
		  	for _, o := range opts {
		  		o(options)
		  	}
		  	return options
		  }
		  
		  // Info logs a message at info level.
		  func (h *Helper) Info(a ...any) {
		  	if !h.Enabled(LevelInfo) {
		  		return
		  	}
		  	_ = h.logger.Log(LevelInfo, h.msgKey, h.sprint(a...))
		  }
		  
		  // Infof logs a message at info level.
		  func (h *Helper) Infof(format string, a ...any) {
		  	if !h.Enabled(LevelInfo) {
		  		return
		  	}
		  	_ = h.logger.Log(LevelInfo, h.msgKey, h.sprintf(format, a...))
		  }
		  ```
	- `Helper` 封装了底层的 `Logger`，使日志记录变得非常简单。你只需传入要记录的内容，`Helper` 会自动设置日志级别和 `message` 键，并调用底层的 `Logger` 完成日志输出。
	- **如何使用 `Helper`？**
		- 使用 `log.NewHelper()` 将一个 `Logger` 实例包装起来即可。
		- ```go
		  import "github.com/go-kratos/kratos/v2/log"
		  
		  // 1. 创建一个 Logger 实例（此处使用默认 Logger）
		  var myLogger log.Logger = log.DefaultLogger
		  
		  // 2. 使用 NewHelper 包装 Logger
		  h := log.NewHelper(myLogger)
		  
		  // 3. 开始愉快地打日志！
		  h.Info("这是一条 Info 日志")
		  ```
	- **Helper 的三种主要打印方法：**
		- **直接输出（如 `Info`，`Debug`）**：
			- 将日志内容作为默认键 `msg` 的值输出，适合简单的日志记录。
			- ```go
			  h.Info("用户下单成功")
			  // 输出示例（JSON 格式）: {"level":"INFO", "msg":"用户下单成功"}
			  ```
		- **格式化输出（以 `f` 结尾，如 `Infof`, `Errorf`）**：
			- ```go
			  userID := 1001
			  h.Infof("用户 %d 查询了商品信息", userID)
			  // 输出: {"level":"INFO", "msg":"用户 1001 查询了商品信息"}
			  ```
		- **结构化输出（以 `w` 结尾，如 `Infow`、`Errorw`）**：
			- 用于记录结构化日志，支持自定义多个键值对字段，便于日志检索与分析。
			- ```go
			  h.Infow(
			      "event", "user_login",
			      "username", "kratos",
			      "ip_address", "127.0.0.1",
			  )
			  // 输出: {"level":"INFO", "event":"user_login", "username":"kratos", "ip_address":"127.0.0.1"}
			  ```
			- 使用 `w` 方法，你的日志将变得结构清晰、语义明确，方便在日志平台中精确查询，例如筛选所有 `username` 为 `"kratos"` 的登录事件。
- **`Valuer`**
	- 用于自动添加全局字段，让每条日志都自动带上统一的上下文信息。
	- **有什么用？**
		- 如果希望每条日志都包含如下一些通用字段：
			- 当前时间（`timestamp`）
			- 服务名称（`service.name`）
			- 实例 ID（`service.id`）
			- 请求追踪 ID（`trace_id`）
			- 调用位置（`caller`）
		- 如果每次都手动写这些信息，将非常繁琐且容易出错。`Valuer` 就是为了解决这个问题。你可以把它理解为一个“值函数”，在每次输出日志时会被自动调用，并将结果附加到日志中。
	- **怎么用？**
		- 通过 `log.With()` 将一个或多个 `Valuer` 绑定到 `Logger` 上，通常在项目启动时初始化日志时配置。
		- ```go
		  // 在 cmd/server/main.go 中
		  logger := log.With(log.NewStdLogger(os.Stdout),
		      "ts", log.DefaultTimestamp,      // 自动添加时间戳
		      "caller", log.DefaultCaller,     // 自动添加调用位置（文件名:行号）
		      "service.id", "my-service-id",   // 固定服务 ID
		      "trace_id", tracing.TraceID(),   // 自动获取 trace_id
		      "span_id", tracing.SpanID(),     // 自动获取 span_id
		  )
		  ```
		- 此后，只要通过这个 `logger` 创建的 `Helper` 打日志，以上字段就会自动附加，无需手动添加。
- `Filter`
	- 用于日志的过滤与脱敏处理。
	- 在生产环境中，你可能不希望打印所有 `Debug` 级别的日志，或者需要对敏感信息（如密码、手机号等）进行打码处理。此时，可以使用 `Filter` 作为日志的“过滤器”。
	- ```go
	  import "github.com/go-kratos/kratos/v2/log"
	  
	  // ... 创建原始 logger ...
	  
	  // 使用 NewFilter 包装 logger
	  filteredLogger := log.NewFilter(logger,
	      // 1. 只输出 Error 级别及以上的日志
	      log.FilterLevel(log.LevelError),
	  
	      // 2. 对 key 为 "password" 或 "token" 的字段进行脱敏处理
	      log.FilterKey("password", "token"),
	  )
	  
	  // 用过滤后的 logger 创建 Helper
	  h := log.NewHelper(filteredLogger)
	  
	  h.Info("这条日志不会被打印") // 因为级别是 Info，低于 Error
	  h.Errorw("event", "login_failed", "password", "123456")
	  // 输出: {"level":"ERROR", "event":"login_failed", "password":"******"}
	  
	  ```
- **Kratos 日志的使用流程：**
	- **初始化**：在 `main` 函数中创建并配置 `Logger` 实例。
	- **依赖注入**：在各层核心结构体（如 `Service`、`Biz` 和 `Data`）中嵌入一个 `*log.Helper` 字段，通过构造函数接收由 Wire 注入的 `Logger`，并使用 `log.NewHelper` 封装后使用。
	- **打印日志**：在具体的业务方法中使用 `log.Helper` 输出日志。
- **日志的最佳实践：**
	- **统一使用 `Helper`**：在业务代码（如 `service`、`biz`、`data` 层）中，通过依赖注入获取 `Logger`，并使用 `log.NewHelper` 进行封装后统一调用。
	- **优先使用结构化日志**：尽量使用以 `-w` 结尾的方法（如 `Infow`），减少使用 `-f` 方法，避免使用无后缀的基本方法。将关键的动态信息作为独立的 `key-value` 字段记录，便于日志平台进行检索与分析。
	- **关联上下文信息是实现日志链路追踪的关键**。通过在各层调用 `log.WithContext(ctx)`，可确保日志绑定当前请求的上下文，使 `TraceID` 能够贯穿整个调用链，便于进行分布式系统中的问题排查。特别是在请求入口（如 `Service` 层），应优先调用 `s.log.WithContext(ctx)`，为后续日志记录奠定基础。
	- **错误日志先记录再返回**：在 `Data` 和 `Biz` 层捕获到错误时，先记录日志，再将错误返回，确保问题不被悄然吞掉。
- **Kratos 三层架构（Service, Biz, Data）下的日志最佳实践：**
	- **`Service` 层（服务层）**
		- **职责**：接收外部请求，进行参数校验（如 DTO 校验），调用 `Biz` 层处理业务逻辑，并将结果封装为对外响应。
		- Service 层的日志是整个请求链路的入口和出口，它的核心目标是记录“我收到了什么，我返回了什么，以及花了多长时间”，作为问题排查的起点。
		- **哪里记？**
			- 通常使用 `logging` 中间件自动化记录，而不是在每个接口方法里手动写。这样可以保证日志格式统一，且业务代码干净。
		- **记什么？**
			- **请求参数**：通常包括请求路径（如 URL 或方法名）和请求体（如 JSON 或表单）中的关键字段。应避免记录体积过大的内容（如图片、文件）或无关数据，重点关注对业务逻辑有实际意义的信息。同时，应对密码、手机号、Token 等敏感字段进行脱敏或过滤，避免泄露用户隐私。
			- **处理耗时**：从接收到请求到返回响应所耗费的总时间。
			- **日志级别**：成功请求使用 `INFO`，失败请求则使用 `WARN` 或 `ERROR`。
	- **`Biz` 层（业务逻辑层）**
		- **职责**：实现核心业务逻辑，编排调用 `Data` 层完成完整的业务流程。它不关心数据来源（数据库、缓存等），也不关注数据最终的返回形式（如协议或格式）。
		- `Biz` 层的日志是排查业务问题的核心依据，其目标是清晰展现业务流程的执行路径和关键决策点。通过记录关键流程的起点、重要的决策分支以及异常情况，使日志成为业务逻辑的“执行说明书”。
		- **在哪记？**
			- 在 UseCase 的关键逻辑执行前手动记录日志，使其清晰反映业务的流转过程。
		- **记什么？**
			- **关键业务流程的入口**：记录接收到的核心业务参数（非原始 DTO，而是经过处理、对业务有意义的字段）。
			- **重要的业务分支或决策点**：在流程进入关键分支时，记录判断条件及选择结果。
			- **调用 `Data` 层的前后**：记录即将调用的接口及其参数，以及调用结果（如有必要）。
			- **业务异常或错误**：
				- 对于可预期的业务异常（如“库存不足”、“优惠券无效”），使用 `WARN` 级别。
				- 对于意料之外的系统错误，使用 `ERROR` 级别。
				- **日志中必须包含足够的上下文信息**，以便快速定位问题。
	- **`Data` 层（数据访问层）**
		- **职责**：负责与具体数据源（如 MySQL、Redis、Kafka、第三方 API 等）交互，提供统一的数据访问接口供 `Biz` 层使用，实现数据的持久化与读取。
		- `Data` 层的日志主要用于排查与外部数据源（如数据库、缓存、第三方 API）交互时出现的问题。其核心目标是记录：“请求了谁、请求了什么、返回了什么、耗时多久”，成为诊断外部依赖问题的“X 光片”。
		- **记什么？**
			- 在执行 I/O 操作前后记录日志。
			- **对外部系统的调用**：
				- **数据库**：记录执行的 SQL 语句及影响的行数。
				- **缓存**：记录访问的 Key，并标明是否命中。
				- **第三方 API**：记录请求的 URL、方法、核心参数和返回的 HTTP 状态码。
				- Kratos 提供的 GORM 插件已集成日志功能，可自动记录 SQL 和耗时等信息，只需在初始化 `gorm.DB` 时传入自定义的 `Logger` 实例。
			- **外部依赖的耗时**：
				- 记录每次 I/O 操作的耗时。若超过设定阈值（如 200ms），可将日志级别提升为 `WARN`，便于及时发现性能瓶颈。
			- **外部依赖返回的错误**：
				- 发生错误（如数据库连接超时、Redis Key 不存在、第三方 API 返回 5xx）时，应记录为 `ERROR` 级别日志。
				- 日志应包含关键上下文信息，如访问的 Key、执行的 SQL、请求的 URL 等，以便快速定位和复现问题。
-