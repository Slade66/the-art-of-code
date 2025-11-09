-
- ## 是什么？有什么用？
	- Zap 是 Uber 开源的高性能日志库。
	- Zap 性能极高，比标准库 `log` 快数倍，开销低；它以结构化的键值对形式输出日志，便于机器处理，同时支持按日志级别分类。
- ## 怎么用？
	- **安装：**`go get -u go.uber.org/zap`
	- **创建日志器（配置 Zap）**
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
		- `defer logger.Sync()` 保证程序退出前将日志写入，在程序退出前调用 `Sync()` 是一个好习惯。
		- 更高级的配置（如日志输出到多个文件、发送到消息队列等）需要直接使用 `go.uber.org/zap/zapcore`。
	- **设置全局日志器**
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
	- **添加上下文字段**
		- ```go
		  // 给当前 logger 添加 request_id 字段
		  reqLogger := logger.With(zap.String("request_id", "abc123"))
		  
		  reqLogger.Info("请求开始")
		  reqLogger.Warn("请求超时", zap.Duration("elapsed", 3*time.Second))
		  
		  ```
		- 用 `logger.With(...)` 创建一个新的 logger，同一请求的日志都会自动包含 `request_id` 字段，便于日志聚合和查询。
-