- **context 的作用：**
  collapsed:: true
	- **传递取消信号：**它是 Go 语言中实现多个 Goroutine 之间同步取消的标准方式。当一个任务不再需要执行时，顶层任务发出“停止”信号，所有衍生的子任务都会接收到并优雅退出，从而避免计算资源的浪费。
		- **超时自动取消：**你可以给任务设置一个“截止时间”。例如，如果数据库查询最多只能执行 3 秒，那么一旦超时，Context 就会发出取消信号，让相关的 Goroutine 立即结束等待，避免某个慢操作拖累整个程序的运行效率。
		- **比喻：**
			- 可以把它想象成在餐厅点餐。从你下单开始，到菜品上桌为止，整个流程都是围绕这一次“点餐请求”展开的。服务员、厨师、传菜员都需要知道这道菜属于哪一桌、哪个订单。如果你突然说“不要了”，他们也需要一种机制能够立刻停下，避免继续浪费时间和食材。
			- 在 Go 中，`context.Context` 就像这个“订单信息”和“取消通知”，负责在处理链路中传递相关数据，并在需要时发出取消信号。
	- **传递请求范围的值（Request-scoped Values）：**
		- 在处理请求时，我们往往需要在整条请求链路中共享一些信息，比如用户 ID。
		- 如何将这些数据传递给深层嵌套的函数？难道要在每个函数的参数列表增加一个 `requestID` 吗？
		- `Context` 提供了一个键值对存储，可以将请求作用域内的数据放入其中，并在需要时取出。这样既不会污染函数签名，又能让链路中的各个环节方便地访问这些与请求相关的元数据，而不必把它们一个个显式传参。
- **注意：**
  collapsed:: true
	- **Context 仅用于传递元数据：**它主要用于在请求链路中携带必要的上下文信息，而不是用来替代函数参数。
	- **Context 应作为函数的第一个参数：**通常命名为 `ctx`，这是 Go 社区普遍遵循的写法。
- `context.Background()`
  collapsed:: true
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
  collapsed:: true
	- 提供一个可以手动停止的 `Context`。当你启动一个或多个 Goroutine，并希望在特定条件下（如操作完成或出错）立即通知所有 Goroutine 停止工作时，使用它。
	- 它返回一个新的 `ctx` 和一个 `cancel` 函数。调用 `cancel()` 会发出“停止”的信号。
- `context.WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)`
  collapsed:: true
	- 提供一个带倒计时的 `Context`。当你需要为某个操作（如数据库查询、API 调用）设置最大执行时间时，它会在时间到达后自动停止操作。
- `context.WithValue(parent Context, key, val any) Context`
  collapsed:: true
	- 会将一组键值对存入新的 `ctx` 中，之后可以通过 `ctx.Value(key)` 取回对应的值。
	- **使用本包定义的非导出的空结构体类型的值作为 Context 的 Key**
		- **防止 Key 冲突：**使用非导出的自定义类型，确保 Key 在全局是类型唯一的，避免与第三方库的字符串 Key 冲突，防止数据被覆盖，并在类型断言时出错。
		- **节省资源：**空结构体 (`struct{}`) 占用 0 字节内存，最大限度地减少了高并发下 Context 创建的内存开销。
- `type Context interface`
	- [[Done() <-chan struct{}]]
	- [[Err() error]]
- **context 的应用：**
  collapsed:: true
	- 在 Kratos 项目中，每层方法的第一个参数都是 `context.Context`，它贯穿始终：
	  collapsed:: true
		- 当 gRPC 或 HTTP 请求到达服务器时，Kratos 框架会为该请求创建一个 `Context`，并在 `Service -> Biz -> Data` 的每一层传递下去。
		- 该 `Context` 携带请求范围的元数据。
		- 如果客户端断开连接或请求超时，`Context` 会被取消。
		- 通过在 `Service -> Biz -> Data` 的每一层传递 `ctx`，可以将“取消信号”和“请求元数据”传递到数据库查询操作，确保在必要时整个处理链能够及时中止。
	- net/http 库在处理每个请求时都会创建一个 `context.Context` 对象，并可以通过 `func (r *Request) Context() context.Context` 方法获取该对象。
-