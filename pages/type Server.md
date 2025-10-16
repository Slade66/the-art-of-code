- **作用：**
	- `http.Server` 结构体用于定义和配置 HTTP 服务器的运行参数，相比便捷函数 `http.ListenAndServe`，它提供了更细粒度的控制，使开发者能够灵活管理服务器行为，如超时设置、TLS 配置和优雅停机，是构建生产级服务的标准方式。
- **结构体定义与字段解析：**
	- ```go
	  type Server struct {
	      Addr string
	      Handler Handler
	      DisableGeneralOptionsHandler bool
	      TLSConfig *tls.Config
	      ReadTimeout time.Duration
	      ReadHeaderTimeout time.Duration
	      WriteTimeout time.Duration
	      IdleTimeout time.Duration
	      MaxHeaderBytes int
	      TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
	      ConnState func(net.Conn, ConnState)
	      ErrorLog *log.Logger
	      BaseContext func(net.Listener) context.Context
	      ConnContext func(ctx context.Context, c net.Conn) context.Context
	      // ... 其他未导出的内部字段
	  }
	  ```
	- `Addr string`：可选字段，用于指定服务器监听的 TCP 地址，格式为 `"host:port"`。若留空，默认监听 `:http`（即 80 端口）。
	- `Handler Handler`：指定处理所有传入请求的处理器。若为 `nil`，则默认使用全局的 `http.DefaultServeMux`。在生产环境中，建议显式创建并指定一个 `mux`，以避免全局状态带来的副作用。
	- `ReadTimeout time.Duration`：设置从连接建立起，到请求体完全读取完成的最大时长，覆盖了从接收请求头到读取完整请求体的整个过程。若为零或负值，则表示不限制超时。
	- `WriteTimeout time.Duration`：设置从请求头读取结束起，到响应写入完成的最大时长。它可防止因客户端读取过慢或处理器耗时过长而导致连接长期占用。
	- `IdleTimeout time.Duration`：在启用 `Keep-Alives` 时，用于设置连接在空闲状态下（即处理完一个请求后，等待下一个新请求到来之前）的最大存活时间。这有助于及时释放不活跃的连接，避免资源浪费。
	- `ReadHeaderTimeout time.Duration`：设置服务器读取请求头的最大时长。相比 `ReadTimeout`，它提供更精细的控制。较短的 `ReadHeaderTimeout` 可以快速拒绝恶意或慢速客户端（如 Slowloris 攻击），而 `ReadTimeout` 则可设为更长或不限，以支持大文件上传等场景。
	- `ErrorLog *log.Logger`：指定一个日志记录器，用于输出运行过程中的错误，如连接失败或处理器异常。若为 `nil`，则默认使用 `log` 包的全局标准日志记录器。
- **注意：**
	- **必须设置超时以保证服务健壮性：** 一个未配置任何超时的 `http.Server` 是非常脆弱的。恶意的或缓慢的客户端可以长时间占用连接，最终耗尽服务器资源。在生产环境中，强烈建议根据业务场景合理配置 `ReadHeaderTimeout`, `WriteTimeout` 和 `IdleTimeout`，这是保护服务器稳定性的第一道防线。
	- **`ReadTimeout` vs `ReadHeaderTimeout` 的选择：**对于需要处理文件上传等可能长时间读取请求体的服务，应优先使用 `ReadHeaderTimeout` 来快速过滤无效连接，而不是设置一个非常长的全局 `ReadTimeout`。这提供了更好的灵活性和安全性。
	- **实现优雅停机：**在更新或重启服务时，直接终止进程会导致正在处理的请求失败，影响用户体验。应该通过监听操作系统信号（如 `syscall.SIGINT`, `syscall.SIGTERM`），并调用 `server.Shutdown(ctx)` 方法来实现优雅停机。该方法会平滑地关闭服务器：首先停止接受新连接，然后等待当前正在处理的请求在指定的 `context` 超时前完成。
	- **`Handler` 应独立创建：**避免直接使用 `http.DefaultServeMux`，因为它是一个全局变量，任何包都可以注册路由，容易导致命名冲突和安全问题。最佳实践是使用 `http.NewServeMux()` 创建一个独立的 `mux` 实例，并将其赋给 `Server` 的 `Handler` 字段。
-