- **为什么我们需要日志？**
	- 日志并不仅仅是 `fmt.Println` 的替代品。在一个专业的后端服务中，日志是系统的“黑匣子”，是实现可观测性（Observability）的基石。
	- **作用：**
		- **调试与排错**：这是最基本的功能。当线上服务出现问题时，日志是我们回溯问题的唯一途径。详尽的日志能够帮助我们了解程序执行的逻辑、变量的值以及错误发生的环节。
		- **监控与告警**：通过分析日志流，我们可以监控服务的压力状况。例如，可以统计单位时间内的系统性能，当指标异常升高时，自动触发告警，提醒开发人员介入。
		- **审计与分析**：日志可以记录关键的用户行为或系统事件，如用户登录、下单等操作，供事后审计或数据分析，帮助了解业务的增长情况。
- **Go 的标准库提供了两个主要的日志包：**
	- **`log`**：这是 Go 早期版本就有的传统日志包，功能简单，主要用于输出自由格式的文本日志。
	- **`log/slog`**：这是 Go 1.21 版本新增的现代化日志包，专门用于输出结构化日志，如 JSON 格式。
- **`slog` 包：**
	- **为什么需要结构化日志？**
		- 假设你需要在成千上万条日志中查找所有“用户 ID 为 123”且“操作失败”的记录。
		- **文本日志**：`[ERROR] User 123 failed to update profile.` 你需要用复杂的正则表达式去匹配，这样既低效又容易出错。
		- **结构化日志（JSON）**：`{"level":"ERROR", "msg":"update profile failed", "user_id":123}`。你可以直接用专门的日志工具解析并查询 `user_id == 123 && level == "ERROR"`，操作既快速又精准。
	- **级别（Level）**：`slog` 支持日志级别（如 `Debug`、`Info`、`Warn`、`Error`），你可以设置最低级别，低于该级别的日志将不会被输出。
	- **`slog` 的三大核心组件：**
		- **`Logger`（记录器）**：最终调用的对象，提供 `.Info()`、`.Warn()`、`.Error()` 等方法。
		- **`Handler`（处理器）**：决定日志的格式（如 `JSON`）和输出位置。
		- **`Attr`（属性）**：一个键值对（Key-Value Pair），是结构化数据的基本单元。
	- **示例：**
		- ```go
		  // 1. 创建一个 Handler，使用 JSON 格式并输出到标准错误
		  jsonHandler := slog.NewJSONHandler(os.Stderr, nil)
		  // 2. 基于该 Handler 创建一个 Logger
		  myslog := slog.New(jsonHandler)
		  // 3. 使用 Logger 记录日志
		  myslog.Info("hi there") // 使用 Info 级别记录一条简单的消息。
		  myslog.Info("hello again", "key", "val", "age", 25) // 除了基本消息，还附带了两对键值对属性
		  ```
- **`log` 包：**
	- **缺点：**
		- **难以解析**：纯文本日志不适合机器处理。
		- **缺乏级别**：没有 `Info`、`Warning`、`Error` 等级别的区分。
	- **基本用法：**
		- `Println()` / `Printf()`：用于记录普通信息，打印日志后程序继续执行。
		- `Fatalf()`：打印日志后，会调用 `os.Exit(1)`，导致程序立即退出。
		- `Panicf()`：打印日志后，会触发一个 `panic`。
	- **配置：**
		- **输出目标：**
			- 默认情况下，标准日志记录器将日志输出到标准错误流（`os.Stderr`），并自动添加日期和时间。你可以通过 `log.SetOutput()` 将日志重定向到任何实现了 `io.Writer` 接口的对象，例如文件或内存中的变量。
			- ```go
			  package main
			  
			  import (
			  	"bytes"
			  	"log"
			  	"os"
			  )
			  
			  func main() {
			  	// 将日志输出重定向到文件
			  	file, _ := os.OpenFile("logfile.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0644)
			  	defer file.Close()
			  	log.SetOutput(file)
			  	log.Println("This is a log message written to the file.")
			  
			  	// 将日志输出重定向到内存中的缓冲区
			  	var buf bytes.Buffer
			  	log.SetOutput(&buf)
			  	log.Println("This is a log message written to memory.")
			  
			  	// 打印内存中的日志内容
			  	log.Println("Logged content:", buf.String())
			  }
			  
			  ```
		- **格式标志：**
			- 你可以通过 `log.SetFlags()` 来控制每条日志开头的前缀信息，如日期、时间、文件名和代码行号等。
			- **示例：**
				- ```go
				  log.SetFlags(log.LstdFlags | log.Lmicroseconds)
				  log.Println("with micro")
				  // 2023/08/22 10:45:16.904141 with micro
				  
				  log.SetFlags(log.LstdFlags | log.Lshortfile)
				  log.Println("with file/line")
				  // 2023/08/22 10:45:16 logging.go:40: with file/line
				  ```
				- `log.LstdFlags` 是 `log.Ldate | log.Ltime` 的简写，表示“日期+时间”。
				- `| log.Lmicroseconds`：按位或操作符 `|` 用于在标准标志的基础上增加微秒级的精度。
				- `| log.Lshortfile`：在标准标志的基础上增加输出日志的文件名和行号，这对调试非常有用。
		- **自定义 Logger：**
			- 除了使用全局标准日志记录器，你还可以通过 `log.New()` 创建自己的日志记录器实例。这使你能够将不同模块的日志输出到不同的位置或使用不同的格式。
			- ```go
			  mylog := log.New(os.Stdout, "my:", log.LstdFlags)
			  mylog.Println("from mylog")
			  // my:2023/08/22 10:45:16 from mylog
			  
			  mylog.SetPrefix("ohmy:")
			  mylog.Println("from mylog")
			  // ohmy:2023/08/22 10:45:16 from mylog
			  ```
			- 第一个参数 `os.Stdout` 指定了输出目标为标准输出。
			- 第二个参数 `"my:"` 设置了固定的前缀。
			- 第三个参数 `log.LstdFlags` 指定了日志的格式。
			- `mylog.SetPrefix()` 允许动态修改记录器的前缀。
-