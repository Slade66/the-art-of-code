- **Kratos 的接口抽象：**
	- Kratos 框架本身不关心你用的是 Consul 还是 Nacos。它提供了一套统一的接口，把“注册”和“发现”这两个动作标准化了。
	- 也就是说，无论使用哪个配置中心，Kratos 都只需调用 Register 方法进行注册，再调用 GetService 方法获取服务实例。
- `type Registrar interface`
	- 这个接口是注册器，“写”电话本，只管“登记”。
	- `Register(ctx, service)`：**注册**。告诉注册中心“我来了”。
	- `Deregister(ctx, service)`：**反注册**。告诉注册中心“我走了”。
- `type Discovery interface`
	- 这个接口是发现器 ，“读”电话本，只管“查询”。
	- `GetService(ctx, serviceName)`：**获取服务**。
		- “你好，请立即告诉我 order-service（服务名）当前所有可用的实例列表。”
		- 这是一种一次性拉取（Pull）操作。
	- `Watch(ctx, serviceName)`：**订阅服务**。
		- “你好，请告诉我 order-service 的实例列表，并且如果这个列表之后发生变化（比如有新实例上线或旧实例下线），请立即通知我。”
		- 这是一种订阅操作，比 GetService 更高级，也更常用，因为它能动态感知服务的变化。
- **Kratos 官方的 contrib 库提供了一系列注册中心插件：**
	- 这些插件的作用相当于“适配器”（Adapter）：
		- 它们实现了 Kratos 定义的 `Registrar` 和 `Discovery` 接口。
		- 它们在内部封装了各自注册中心（如 Consul、Etcd）的原生 Go SDK（Native SDK），负责将 Kratos 的标准调用（例如 Register）转换为对应注册中心 SDK 的 API 调用。
- **使用方式：**
	- **注册服务实例**
		- 你只需创建一个官方 SDK 的客户端，基于它生成一个 `Registrar` 实例，并将该实例注入到 `kratos.New()` 中。整个过程中，无需手动调用 `reg.Register()` 或 `reg.Deregister()`。
		- **核心步骤：**
			- **创建注册中心的官方 SDK 客户端。**
			  logseq.order-list-type:: number
			- **创建 `Registrar` 实例：**使用 Kratos 对应的注册中心插件，将上一步创建的原生客户端进行包裹适配。
			  logseq.order-list-type:: number
			- **注入注册器：**将 `kratos.Registrar(reg)` 传入 `kratos.New()`。
			  logseq.order-list-type:: number
		- **Kratos 的自动化生命周期管理：**
			- 当你调用 `app.Run()` 时，Kratos 会自动执行`reg.Register(ctx, serviceInstance)` ，将当前服务实例注册到注册中心。
			- 当你按下 `Ctrl+C` 触发优雅关停时，Kratos 会自动调用 `reg.Deregister(ctx, serviceInstance)`，通知注册中心该实例即将下线（从“电话本”中移除），然后再依次关闭 HTTP 和 gRPC 服务器。
		- **代码示例：**
			- ```go
			  ```
-