- 在 Kratos 项目中，每个服务方法的第一个参数都是 `context.Context`，它贯穿始终：
	- 当 gRPC 或 HTTP 请求到达服务器时，Kratos 框架会为该请求创建一个 `Context`，并在 `Service -> Biz -> Data` 的每一层传递下去。
	- 该 `Context` 携带请求的元数据（如 `trace_id`）。
	- 如果客户端断开连接或请求超时，`Context` 会被取消。
	- 通过在 `Service -> Biz -> Data` 的每一层传递 `ctx`，可以将“取消信号”和“请求元数据”传递到数据库查询操作，确保在必要时整个处理链能够及时中止。
- **类比：**
	- 想象一下你在餐厅点餐的场景，从你下单那一刻起，直到菜品上桌，整个过程都围绕着你这次“点餐请求”进行。服务员、厨师、传菜员都需要知道这道菜是属于哪一桌、哪个订单的。如果中途你突然说“我不要了”，他们也需要一个机制来立刻停下手中的活，避免浪费。
	- 在 Go 语言中，`context.Context` 就扮演着这个“订单信息”和“取消通知”的角色。
- **`context.Context` 的三大核心作用：**
	- **取消信号：**它是 Go 语言中实现多个 Goroutine 之间同步取消的标准方式。可以将其比作一个信号灯，当一个任务不再需要时，顶层任务发出“停止”信号，所有衍生的子任务都会接收到并优雅退出，从而避免计算资源的浪费。
	- **超时自动取消：**你可以为任务设定一个“截止时间”。比如，若数据库查询的最大执行时间为 3 秒，超出这个时间后，`Context` 会自动发送“取消”信号，相关的 Goroutine 会立即停止等待，防止单个慢操作影响整个程序的运行效率。
	- **传递请求范围的值（Request-scoped Values）：**
		- 在请求处理过程中，常常需要共享一些数据，例如用户的 ID。
		- 如何将这些数据传递给深层嵌套的函数？难道每个函数都要增加一个 `requestID` 参数吗？这样会让函数签名变得一团糟。
		- `Context` 提供了一个键值对存储，可以将“请求范围”的数据存入 `Context`，并在需要时读取。这样既避免了污染函数签名，又能方便地在请求处理链中共享与请求相关的元数据，无需显式地将这些值作为函数参数传递。
- **注意：**
	- `context.Value` 只应该用来传递请求传递范围的元数据，而不是函数的参数。
	- 同一个 `Context` 可以传递给多个 Goroutine，它是并发安全的。
	- 不要传递一个 `nil` 的 `Context`。如果你不确定用什么，就用 `context.TODO()`。
	- 不要将 `Context` 存储在结构体中，应该显式地在函数间传递。
	- `Context` 应该永远作为函数的第一个参数，并且通常命名为 `ctx`。
- **API：**
	- `context.Background()`
		- 创建一个全新的、干净的、与任何现有任务都无关的“根” `Context`，可以理解为“程序的全局背景板”。当你需要一个“祖先” `Context` 作为所有 `Context` 派生链的起点时，就使用它。
		- **将它传入其它创建子`Context`的函数：**
			- ```go
			  ctx, cancel := context.WithTimeout(context.Background(), 2 * time.Second)
			  ```
			- 这行代码的意思是：“请在`context.Background()`这块干净的‘背景板’上，创建一个新的子任务节点。这个子任务有一个特殊属性：必须在2秒内完成，否则就自动取消。”
			- `WithTimeout(parent, ...)` 用于创建一根树枝，你需要指定它附加在哪根“树干”（`parent`）上。对于一个全新的顶层超时任务，它的“树干”自然是 `Background()` 这个“大地”本身。
		- **树根的特点：**
			- 它不是从任何其他任务派生出来的：它是独立存在的，代表一个新任务的开始。
			- 它自己永远不会“死”：没有过期时间，也不能被外部取消。只有当整个程序结束时，它才会消失。
		- **级联取消（连坐）：**
			- 将 `Context` 想象成一棵“任务树”，整个 Go 程序就是一棵庞大的任务树。任务可以分解为多个子任务，子任务可以继续分解……形成一个树状结构。
			- 当一个 `Context` 被取消时，所有从它派生的 `Context` 也会被一并取消。就像砍掉树枝，所有挂在这根树枝上的叶子也会随之凋零。
	- `context.WithCancel(parent Context) (ctx Context, cancel CancelFunc)`
		- 提供一个可以手动停止的 `Context`。当你启动一个或多个 Goroutine，并希望在特定条件下（如操作完成或出错）立即通知所有 Goroutine 停止工作时，使用它。
		- 它返回一个新的 `ctx` 和一个 `cancel` 函数。调用 `cancel()` 会发出“停止”的信号。
	- `context.WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)`
		- 提供一个带倒计时的 `Context`。当你需要为某个操作（如数据库查询、API 调用）设置最大执行时间时，它会在时间到达后自动停止操作。
	- `context.WithValue(parent Context, key, val any) Context`
		- 会将一组键值对存入新的 `ctx` 中，之后可以通过 `ctx.Value(key)` 取回对应的值。
		- `context` 主要用于在整个请求处理链中传递与请求范围相关的元数据，而不是普通的函数参数。
		- Go 标准库文档建议在包内定义未导出的自定义类型作为 key，以避免冲突。因为你无法预知第三方库是否会使用相同的 key，一旦冲突，可能导致你写入的数据被覆盖，后续取值时的类型断言就会出错甚至导致程序崩溃。
- **例子：**
	- ```go
	  func hello(w http.ResponseWriter, req *http.Request) {
	      // (1) 获取由 net/http 框架为每个请求创建的 Context
	      ctx := req.Context()
	      fmt.Println("server: hello handler started")
	      defer fmt.Println("server: hello handler ended")
	  
	      // (2) 使用 select 监听两个 channel
	      select {
	      case <-time.After(10 * time.Second): // 正常情况
	          fmt.Fprintf(w, "hello\n")
	      case <-ctx.Done(): // 被取消的情况
	          // (3) 获取取消的原因
	          err := ctx.Err()
	          fmt.Println("server:", err)
	          http.Error(w, err.Error(), http.StatusInternalServerError)
	      }
	  }
	  ```
	- **`hello` 处理器函数分析：**
		- **第 1 步：**`ctx := req.Context()`
			- Go 的 `net/http` 服务器非常智能，它会在接收到每个 HTTP 请求时，自动为该请求创建一个 `Context`，并将其附加到 `http.Request` 对象上。我们通过 `.Context()` 方法就能拿到它。这个 `ctx` 的生命周期与这个请求绑定。
		- **第 2 步：**`select` 语句
			- 这里的 `select` 在同时等待两个事件，哪个先发生就执行哪个分支。
			- `case <-time.After(10 * time.Second)`：这是一个模拟耗时操作。`time.After` 返回一个 channel，它会在 10 秒后发送一个值。如果 10 秒内没有任何“取消”信号，这个分支就会被执行，服务器会正常响应 "hello\n"。
			- `case <-ctx.Done()`：`ctx.Done()` 返回一个 channel。这个 channel 在正常情况下会一直阻塞，但一旦这个 `Context` 被取消（比如客户端断开连接），Go 就会立即关闭这个 channel。从一个已关闭的 channel 读取数据会立刻返回，所以这个 case 会被选中。
		- **第 3 步：`err := ctx.Err()`**
			- 当 `ctx.Done()` 被触发后，我们可以通过 `ctx.Err()` 来获取取消的原因。它通常会返回两个预定义错误之一：`context.Canceled` 或 `context.DeadlineExceeded`。
	- 当服务器端的 `net/http` 框架检测到客户端连接已断开时，它认为该请求不再有效，于是取消与该请求绑定的 `Context`。这一“取消”操作会导致服务器端的 `ctx.Done()` channel 被关闭，从而触发 `select` 的第二个 case。
-