- **作用：**
	- `http.Transport` 是 Go 语言 `http.Client` 背后真正执行请求和管理连接的引擎。它通过提供强大的可配置性，让开发者能够精确地控制底层网络行为，从而满足各种复杂的应用场景需求。
	- `Transport` 的核心功能主要包括：
		- **连接池：**`Transport` 会维护一个底层 TCP 连接池。HTTP 请求完成后，连接不会立即关闭，而是被置为空闲状态，供同一服务器的后续请求复用。这减少了 TCP 三次握手和 TLS 握手的开销，从而显著提升性能。
		- **并发控制：**`Transport` 可以限制与同一主机的最大并发连接数 (`MaxConnsPerHost`) 和最大空闲连接数 (`MaxIdleConnsPerHost`)，以避免资源耗尽或对服务器造成过大压力。
		- **底层网络连接：**`Transport` 通过 `DialContext` 或 `DialTLSContext` 等字段来建立并管理底层网络连接。开发者可以根据需要自定义连接方式，例如指定网络协议或实现自定义的超时逻辑。
		- **代理支持：**`Transport` 支持通过 `Proxy` 字段配置 HTTP/HTTPS 代理，将请求转发到指定的中间服务器。
		- **TLS/SSL 配置：**`TLSClientConfig` 字段提供完整的 TLS 配置选项，包括证书验证、客户端证书和 TLS 版本等，确保 HTTPS 连接的安全性。
		- **超时控制：**`ResponseHeaderTimeout` 等字段可以精细化控制请求各阶段的超时时间，防止请求无限期阻塞。
- **结构体定义：**
	- ```go
	  type Transport struct {
	  	Proxy                  func(*Request) (*url.URL, error)
	  	OnProxyConnectResponse func(ctx context.Context, proxyURL *url.URL, connectReq *Request, connectRes *Response) error
	  	DialContext            func(ctx context.Context, network, addr string) (net.Conn, error)
	  	Dial                   func(network, addr string) (net.Conn, error) // Deprecated
	  	DialTLSContext         func(ctx context.Context, network, addr string) (net.Conn, error)
	  	DialTLS                func(network, addr string) (net.Conn, error) // Deprecated
	  	TLSClientConfig        *tls.Config
	  	TLSHandshakeTimeout    time.Duration
	  	DisableKeepAlives      bool
	  	DisableCompression     bool
	  	MaxIdleConns           int
	  	MaxIdleConnsPerHost    int
	  	MaxConnsPerHost        int
	  	IdleConnTimeout        time.Duration
	  	ResponseHeaderTimeout  time.Duration
	  	ExpectContinueTimeout  time.Duration
	  	TLSNextProto           map[string]func(authority string, c *tls.Conn) RoundTripper
	  	ProxyConnectHeader     Header
	  	GetProxyConnectHeader  func(ctx context.Context, proxyURL *url.URL, target string) (Header, error)
	  	MaxResponseHeaderBytes int64
	  	WriteBufferSize        int
	  	ReadBufferSize         int
	  	ForceAttemptHTTP2      bool
	  	HTTP2                  *HTTP2Config
	  	Protocols              *Protocols
	  }
	  ```
- **重要字段解析：**
	- `Proxy func(*Request) (*url.URL, error)`
		- **解释：**
			- 该字段是一个函数类型，用于为每个发出的请求动态地指定代理服务器。`Transport` 在执行每个请求前，都会调用此函数来决定网络路径。你可以根据请求的特征（如目标主机、URL 路径等）来决定使用哪个代理，或者不使用代理。如果函数返回 `(nil, nil)`，则代表请求将直接连接到目标服务器，不经过任何代理。Go 的 `net/http` 包原生支持 `http`、`https` 和 `socks5` 方案的代理。
	- `DialContext func(ctx context.Context, network, addr string) (net.Conn, error)`
		- **解释：**
			- 此字段允许你完全接管创建底层网络连接的过程。`Transport` 会调用你提供的函数来获取一个满足 `net.Conn` 接口的连接实例，而不再自行建立标准的 TCP 或 Unix Socket 连接。这个强大的功能点允许你将 HTTP 通信嫁接到任何自定义的传输层之上，例如 SSH 隧道或内存管道。通过它，你可以精细地控制连接超时、TCP keep-alive 等网络层面的参数。`net.Dialer` 结构体是实现此函数的常用辅助工具。
	- `TLSClientConfig *tls.Config`
		- **解释：**
			- 此字段用于为 HTTPS 请求配置所有 TLS/SSL 相关的客户端行为。通过它可以完全自定义安全连接的握手过程和策略。常见的用途包括设置 `InsecureSkipVerify: true` 来在测试环境中跳过不安全的证书验证。它同样也用于加载自定义的根证书颁发机构(CA)，以便信任私有签发的证书。
	- `TLSHandshakeTimeout time.Duration`
		- **解释：**
			- 该字段为建立 HTTPS 连接时的 TLS 握手阶段设置一个独立的超时。此超时专门针对客户端与服务器进行加密协商的过程，可以有效防止因对方服务无响应或网络问题导致程序在安全握手阶段无限期阻塞。如果设置为零，则表示没有超时限制。
	- `DisableKeepAlives bool`
		- **解释：**
			- 该字段是一个布尔值，如果设置为 `true`，则会禁用 HTTP 的长连接（Keep-Alive）功能。禁用后，每个 HTTP 请求都会强制建立一个新的 TCP 连接，并在请求完成后立即关闭，连接将不会被回收复用。这通常会严重影响性能，但在某些特殊场景下，如需要规避行为异常的负载均衡器或确保请求之间完全隔离时，可能会用到。
	- `MaxIdleConns int`
		- **解释：**
			- 该字段控制整个 Transport 实例的连接池所能容纳的空闲连接总数上限。这是一个全局性限制，有助于从整体上管理客户端的资源消耗，防止因空闲连接过多而占用大量内存和文件描述符。若设置为零，则表示不设上限。
	- `MaxIdleConnsPerHost int`
		- **解释：**
			- 该字段为连接池针对单个主机（如 `api.example.com`）可以保持的空闲连接数设定上限。这是一个更细粒度的限制，在客户端与多个不同主机通信时尤其有用。它能有效防止某个访问频繁的主机独占所有空闲连接，从而确保连接资源在不同主机间得到更公平的分配。
	- `IdleConnTimeout time.Duration`
		- **解释：**
			- 该字段设定了空闲连接在连接池中的最长存活时间。若一个连接在此期间未被任何新请求复用，Transport 将自动将其关闭并从池中移除。这不仅能主动回收资源，还能有效避免因防火墙或负载均衡器静默断开长时间空闲的连接而导致的请求错误。
	- `ResponseHeaderTimeout time.Duration`
		- **解释：**
			- 该字段用于设置一个精确的超时，其计时周期从客户端完全发送完请求（包括请求体）开始，到成功接收到服务器响应头为止，不包含连接、握手或读取响应体的时间。此设置是提升客户端健壮性的关键，能有效防止因下游服务处理缓慢或“假死”而导致进程永久阻塞。
- **注意：**
	- **必须复用 `Transport`：**`Transport` 是并发安全的，内部维护连接池。切勿为每个请求新建一个 `Transport`，否则连接无法复用，会产生大量处于 `TIME_WAIT` 的 TCP 连接，最终拖垮系统资源。正确做法是在应用内创建并复用一个全局/共享的 `http.Client`（其内部包含 `Transport`）。
	- **`http.Client` 的 `Timeout` 与 `Transport` 的关系：**`Client.Timeout` 是“端到端总超时”，覆盖从建连、发请求到读完整个响应体的全过程；`Transport` 提供更细粒度的超时控制，例如 `DialContext`（连接超时）、`TLSHandshakeTimeout`（TLS 握手超时）、`ResponseHeaderTimeout`（等待响应头超时）。小比喻：把 `Client.Timeout` 当作整场比赛的总计时器，`Transport` 的各项超时是每个分段的小计时器。
	- **谨慎修改全局的 `http.DefaultTransport`：**Go 提供全局的 `http.DefaultTransport` 与 `http.DefaultClient`。直接改动 `http.DefaultTransport`（如调整 `TLSClientConfig`）会影响所有使用 `http.DefaultClient` 的代码，包括第三方库，容易产生隐蔽副作用。最佳实践是为你的应用单独创建自定义的 `Transport` 与 `Client`，而不是修改全局默认值。