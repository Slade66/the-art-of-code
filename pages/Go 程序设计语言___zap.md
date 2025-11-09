-
- ## 是什么？有什么用？
	- Zap 是 Uber 开源的高性能日志库。
	- Zap 性能极高，比标准库 `log` 快数倍，开销低；它以结构化的键值对形式输出日志，便于机器处理，同时支持按日志级别分类。
- ## 怎么用？
	- **安装：**`go get -u go.uber.org/zap`
	- **创建日志器（配置 Zap）**
	  collapsed:: true
		- 最简单的方式是使用 Zap 提供的预设函数：`NewExample`、`NewProduction` 和 `NewDevelopment`：
			- ```go
			  // 开发环境（格式化输出、带颜色、易读）
			  logger, _ := zap.NewDevelopment()
			  
			  // 生产环境（JSON 输出，结构化，性能更好）
			  logger, _ := zap.NewProduction()
			  
			  // 示例（最简配置）
			  logger := zap.NewExample()
			  defer logger.Sync() // 刷新缓冲区
			  
			  ```
		- **`defer logger.Sync()`：**
			- Zap 的日志器底层是异步写入的，也就是说日志先写入内存缓冲区，不是立即落盘（或打印）。
			- 保证程序退出前将缓冲区里的日志刷新（flush）到输出（文件、控制台等），在程序退出前调用 `Sync()` 是一个好习惯，保证最后那几条日志不会因为程序退出而丢失。
		- **`zap.NewDevelopment()` vs `zap.NewProduction()`**
			- `zap.NewDevelopment()`
				- 调试用，看日志像平时的 `fmt.Println` 一样直观，更易读的文本格式（人看得懂），默认 Debug 级别（输出详细信息）。
				- `2025-11-09T14:03:21.123+0800 DEBUG main.go:10 Something happened`
			- `zap.NewProduction()`
				- 上线后用，输出 JSON 给日志收集系统（如 ELK、Prometheus、Grafana Loki 等），结构化 JSON 格式（机器解析友好），默认 Info 级别（跳过调试信息）。
				- `{"level":"info","ts":1731138201.123,"caller":"main.go:10","msg":"Something happened"}`
	- **设置全局日志器**
	  collapsed:: true
		- **痛点：**如果不设置全局日志器，你每次都要在每个组件、函数之间显式传递 `logger`。
		- **办法：**
			- 把你创建的 `logger` 设为 Zap 的全局默认日志器，以后可以在任意位置直接使用 `zap.L()` 或 `zap.S()` 获取全局 logger。
			- ```go
			  zap.ReplaceGlobals(logger)
			  zap.L().Info("全局日志器输出")
			  zap.S().Infow("全局日志器示例", "module", "main")
			  ```
		- **好处：**
			- **减少依赖传递**：不必在函数参数里传 logger，代码更简洁。
			- **日志格式和配置统一**：整个系统使用同一个 logger，输出格式、级别、字段都一致。
	- **SugaredLogger（推荐在一般场景使用，便捷写法）**
	  collapsed:: true
		- 在性能有要求但不是关键的场景中，使用 SugaredLogger。
			- 它比其他结构化日志库快 4–10 倍；
			- 支持结构化日志和 printf 风格日志；
			- 可接受任意数量的键值对。
		- ```go
		  sugar := logger.Sugar()
		  
		  // 结构化日志
		  sugar.Infow("failed to fetch URL",
		    "url", "http://example.com",
		    "attempt", 3,
		    "backoff", time.Second,
		  )
		  
		  // printf 风格日志
		  sugar.Infof("failed to fetch URL: %s", "http://example.com")
		  
		  ```
	- **Logger（推荐在性能敏感场景使用）**
	  collapsed:: true
		- 在极端性能要求的场景下，使用 Logger。
			- 它比 SugaredLogger 更快；
			- 分配更少；
			- 只支持强类型、结构化日志。
		- ```go
		  logger, _ := zap.NewProduction()
		  defer logger.Sync()
		  
		  logger.Info("服务启动")                            // 打印普通信息
		  logger.Warn("缓存未命中")                          // 打印警告
		  logger.Error("操作失败", zap.String("user_id", "123"), zap.Duration("backoff", time.Second), zap.Error(err)) // 打印错误 + 结构化字段
		  
		  ```
		- `zap.String("status", "ok")` 是结构化字段，用来添加上下文信息。
	- **日志打印函数**
	  collapsed:: true
		- ```go
		  Debug(msg string, fields ...zap.Field)	打印调试级别日志
		  Info(msg string, fields ...zap.Field)	打印普通信息日志
		  Warn(msg string, fields ...zap.Field)	打印警告日志
		  Error(msg string, fields ...zap.Field)	打印错误日志
		  DPanic(msg string, fields ...zap.Field)	开发模式下会 panic
		  Panic(msg string, fields ...zap.Field)	打印日志后触发 panic
		  Fatal(msg string, fields ...zap.Field)	打印日志后立即调用 os.Exit(1) 退出程序
		  ```
		- 这些方法都接受：
			- 一个字符串消息 `msg`
			- 任意数量的键值对（`zap.Field`）
	- **`zap.Field` 的构造函数**
	  collapsed:: true
		- `zap.Field` 是 Zap 中用于表示**日志字段的键值对**结构。
		- 当你向日志方法中传入 `fields ...zap.Field` 时，Zap 会将这些字段打包成结构化数据并输出为 JSON。
		- 常见的字段构造函数如下：
			- ```go
			  字符串, `zap.String(key, value)`
			  字符串切片, `zap.Strings(key, []string)`
			  整数, `zap.Int(key, value)`
			  整数(int64), `zap.Int64(key, value)`
			  无符号整数(uint64), `zap.Uint64(key, value)`
			  浮点数, `zap.Float64(key, value)`
			  布尔值, `zap.Bool(key, value)`
			  时间, `zap.Time(key, time.Time)`
			  时长, `zap.Duration(key, time.Duration)`
			  错误, `zap.Error(err)`
			  任意类型, `zap.Any(key, value)`
			  二进制, `zap.Binary(key, []byte)`
			  对象, `zap.Object(key, zapcore.ObjectMarshaler)`
			  嵌套结构, `zap.Inline(anyStruct)`
			  命名空间, `zap.Namespace(key)`
			  堆栈信息, `zap.Stack(key)`
			  ```
	- **添加上下文字段**
	  collapsed:: true
		- ```go
		  // 给当前 logger 添加 request_id 字段
		  reqLogger := logger.With(zap.String("request_id", "abc123"))
		  
		  reqLogger.Info("请求开始")
		  reqLogger.Warn("请求超时", zap.Duration("elapsed", 3*time.Second))
		  
		  ```
		- 用 `logger.With(...)` 创建一个新的 logger，同一请求的日志都会自动包含 `request_id` 字段，便于日志聚合和查询。
-