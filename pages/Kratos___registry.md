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
		- 它们在内部封装了各自注册中心（如 Consul、Etcd）的原生 Go SDK（Native SDK），负责将 Kratos 的标准调用（例如 `Register()`）转换为对应注册中心 SDK 的 API 调用。
- **使用方式：**
	- **注册服务实例：**
		- 你只需创建一个官方 SDK 的客户端，基于它生成一个 `Registrar` 实例，并将该实例注入到 `kratos.New()` 中。整个过程中，无需手动调用 `reg.Register()` 或 `reg.Deregister()`。
		- **核心步骤：**
			- **创建注册中心的官方 SDK 客户端。**
			  logseq.order-list-type:: number
			  id:: 68f87aa0-f443-475f-b9eb-ea53684f03bd
			- **创建 `Registrar` 实例：**使用 Kratos 对应的注册中心插件，将上一步创建的原生客户端进行包裹适配。
			  logseq.order-list-type:: number
			- **注入注册器：**将 `kratos.Registrar(reg)` 传入 `kratos.New()`。
			  logseq.order-list-type:: number
		- **Kratos 的自动化生命周期管理：**
			- 当你调用 `app.Run()` 时，Kratos 会自动执行`reg.Register(ctx, serviceInstance)` ，将当前服务实例注册到注册中心。
			- 当你按下 `Ctrl+C` 触发优雅关停时，Kratos 会自动调用 `reg.Deregister(ctx, serviceInstance)`，通知注册中心该实例即将下线（从“电话本”中移除），然后再依次关闭 HTTP 和 gRPC 服务器。
		- **注册到 nacos：**
			- 注册后，在 Nacos 中实际的服务名为 `Name + "." + scheme`，在本示例中是 `test.http` 或 `test.grpc`。
			- **http：**
			  collapsed:: true
				- ```go
				  package main
				  
				  import (
				  	"log"
				  	"net/http"
				  
				  	khttp "github.com/go-kratos/kratos/v2/transport/http"
				  	"github.com/nacos-group/nacos-sdk-go/clients"
				  	"github.com/nacos-group/nacos-sdk-go/common/constant"
				  	"github.com/nacos-group/nacos-sdk-go/vo"
				  
				  	nacosreg "github.com/go-kratos/kratos/contrib/registry/nacos/v2"
				  	"github.com/go-kratos/kratos/v2"
				  )
				  
				  func pingHandler(w http.ResponseWriter, r *http.Request) {
				  	_, _ = w.Write([]byte("pong"))
				  }
				  
				  func main() {
				  	// 1) 创建 HTTP Server
				  	httpSrv := khttp.NewServer(
				  		khttp.Address(":8000"),
				  	)
				  
				  	// 注册路由
				  	httpSrv.Handle("/ping", http.HandlerFunc(pingHandler))
				  
				  	// 2) 创建 Nacos Naming 客户端
				  	sc := []constant.ServerConfig{*constant.NewServerConfig(
				  		"10.30.60.115",
				  		8081,
				  		constant.WithContextPath("/nacos"),
				  		constant.WithScheme("http"),
				  	)}
				  	cc := &constant.ClientConfig{
				  		NamespaceId:         "",
				  		Username:            "nacos",
				  		Password:            "NTcxfz123nacos",
				  		TimeoutMs:           5000,
				  		NotLoadCacheAtStart: true,
				  		LogDir:              "/tmp/nacos/log",
				  		CacheDir:            "/tmp/nacos/cache",
				  		LogLevel:            "debug",
				  	}
				  	cli, err := clients.NewNamingClient(vo.NacosClientParam{ClientConfig: cc, ServerConfigs: sc})
				  	if err != nil {
				  		log.Fatal(err)
				  	}
				  
				  	// 3) 创建 Nacos Registry
				  	r := nacosreg.New(cli)
				  
				  	// 4) 创建 Kratos 应用，注入 Registrar 与 Server
				  	app := kratos.New(
				  		kratos.Name("test"), // 服务基础名
				  		kratos.Server(httpSrv),
				  		kratos.Registrar(r),
				  	)
				  
				  	if err := app.Run(); err != nil {
				  		log.Fatal(err)
				  	}
				  }
				  
				  ```
			- **grpc：**
			  collapsed:: true
				- ```go
				  package main
				  
				  import (
				  	"log"
				  
				  	"github.com/nacos-group/nacos-sdk-go/clients"
				  	"github.com/nacos-group/nacos-sdk-go/common/constant"
				  	"github.com/nacos-group/nacos-sdk-go/vo"
				  
				  	nacosreg "github.com/go-kratos/kratos/contrib/registry/nacos/v2"
				  	"github.com/go-kratos/kratos/v2"
				  	"github.com/go-kratos/kratos/v2/transport/grpc"
				  )
				  
				  func main() {
				  	// 1) 启动一个 gRPC Server
				  	gsrv := grpc.NewServer(grpc.Address(":9000"))
				  
				  	// 2) 创建 Nacos Naming 客户端
				  	sc := []constant.ServerConfig{*constant.NewServerConfig(
				  		"10.30.60.115",
				  		8081,
				  		constant.WithContextPath("/nacos"),
				  		constant.WithScheme("http"),
				  	)}
				  	cc := &constant.ClientConfig{
				  		NamespaceId:         "",
				  		Username:            "nacos",
				  		Password:            "NTcxfz123nacos",
				  		TimeoutMs:           5000,
				  		NotLoadCacheAtStart: true,
				  		LogDir:              "/tmp/nacos/log",
				  		CacheDir:            "/tmp/nacos/cache",
				  		LogLevel:            "debug",
				  	}
				  	cli, err := clients.NewNamingClient(vo.NacosClientParam{ClientConfig: cc, ServerConfigs: sc})
				  	if err != nil {
				  		log.Fatal(err)
				  	}
				  
				  	// 3) 创建 Nacos Registry
				  	r := nacosreg.New(cli)
				  
				  	// 4) 创建 Kratos 应用，注入 Registrar 与 Server
				  	app := kratos.New(
				  		kratos.Name("test"), // 服务基础名
				  		kratos.Server(gsrv),
				  		kratos.Registrar(r),
				  	)
				  	if err := app.Run(); err != nil {
				  		log.Fatal(err)
				  	}
				  }
				  
				  ```
	- **服务发现：**
		- **核心步骤：**
			- 创建 Nacos 原生 Naming 客户端 (`cli`)，用于连接 Nacos Server。
			  logseq.order-list-type:: number
			- 创建 Kratos Nacos 适配器 (`r`)，将 Nacos 原生客户端 `cli` 包装成 Kratos 认识的 `Discovery` 接口。
			  logseq.order-list-type:: number
			- 创建 Kratos HTTP 客户端 (`conn`)，通过 `WithDiscovery(r)` 注入 Nacos 适配器，并通过 `WithEndpoint("discovery:///test.http")` 锁定目标服务名。
			  logseq.order-list-type:: number
			- 使用 `conn` 客户端，直接向相对路径 (`/ping`) 发起请求，Kratos 自动完成服务发现、负载均衡和 URL 重写。
			  logseq.order-list-type:: number
		- **示例代码：**
		  collapsed:: true
			- ```go
			  package main
			  
			  import (
			  	"context"
			  	"io"
			  	"log"
			  	"net/http"
			  
			  	"github.com/nacos-group/nacos-sdk-go/clients"
			  	"github.com/nacos-group/nacos-sdk-go/common/constant"
			  	"github.com/nacos-group/nacos-sdk-go/vo"
			  
			  	nacosreg "github.com/go-kratos/kratos/contrib/registry/nacos/v2"
			  	kratoshttp "github.com/go-kratos/kratos/v2/transport/http"
			  )
			  
			  func main() {
			  	// 1) Nacos Naming 客户端
			  	sc := []constant.ServerConfig{*constant.NewServerConfig(
			  		"10.30.60.115",
			  		8081,
			  		constant.WithContextPath("/nacos"),
			  		constant.WithScheme("http"),
			  	)}
			  	cc := &constant.ClientConfig{
			  		NamespaceId:         "",
			  		Username:            "nacos",
			  		Password:            "NTcxfz123nacos",
			  		TimeoutMs:           5000,
			  		NotLoadCacheAtStart: true,
			  		LogDir:              "/tmp/nacos/log",
			  		CacheDir:            "/tmp/nacos/cache",
			  		LogLevel:            "debug",
			  	}
			  	cli, _ := clients.NewNamingClient(vo.NacosClientParam{ClientConfig: cc, ServerConfigs: sc})
			  
			  	r := nacosreg.New(cli, nacosreg.WithDefaultKind("http"))
			  
			  	conn, err := kratoshttp.NewClient(
			  		context.Background(),
			  		kratoshttp.WithEndpoint("discovery:///test.http"),
			  		kratoshttp.WithDiscovery(r),
			  	)
			  	if err != nil {
			  		log.Fatal(err)
			  	}
			  	defer conn.Close()
			  
			  	// 使用 conn 做 http 请求
			  	req, _ := http.NewRequestWithContext(context.Background(), http.MethodGet, "/ping", nil)
			  	resp, err := conn.Do(req)
			  	if err != nil {
			  		log.Fatal(err)
			  	}
			  	defer resp.Body.Close()
			  	body, _ := io.ReadAll(resp.Body)
			  	log.Println(string(body))
			  }
			  
			  ```
			- 这里没有使用 Go 标准库的 `http.Client`，而是创建了一个由 Kratos 封装的客户端 `kratoshttp.NewClient`。
			- `kratoshttp.WithDiscovery(r)`：这是关键的注入步骤，用来告诉 Kratos 客户端 `conn`，它的服务发现工具是 `r`（Nacos 适配器）。
			- `kratoshttp.WithEndpoint("discovery:///test.http")`：
				- `discovery:///` 这个 Scheme（协议）是 Kratos 的关键字，用于启用服务发现功能。
				- `test.http` 是要连接的逻辑服务名。
				- 执行完这行代码后，`conn` 就成为一个“智能”客户端。它知道目标服务是 `test.http`，并且知道如何通过 `r`（Nacos 适配器）去发现并连接该服务。
			- 当你调用 `conn.Do(req)` 时，Kratos 客户端 `conn` 内部会自动执行以下流程：
				- 检查自身配置，发现通过 `WithEndpoint("discovery:///test.http")` 绑定到了服务 `test.http`。
				- 使用注入的 `r`（Nacos 适配器）向 Nacos 查询：“请返回 `test.http` 服务的健康实例列表。”
				- `r` 调用底层的 `cli` 客户端，Nacos Server 返回一个实例列表，例如 `["10.1.2.3:8000", "10.1.2.4:8000"]`。
				- `conn` 内置的负载均衡器（默认使用 Round Robin）从列表中选择一个实例，如 `10.1.2.3:8000`。
				- 发现请求 URL 是相对路径 `/ping`，于是自动重写请求，将其拼接为绝对路径 `http://10.1.2.3:8000/ping`。
				- 最后，`conn` 使用底层的标准 `http.Client` 向该地址发起真实的网络请求，并返回响应 `resp`。
			- **总结：**
				- 作为开发者，你只需面向服务名（`test.http`）和相对路径（`/ping`）进行编程。Kratos 会通过适配器（`r`）和依赖注入（`WithDiscovery`），自动完成服务发现、负载均衡以及 URL 重写等繁琐的工作。
- **NewNamingClient 和 NewConfigClient 有什么不同？**
	- **NewNamingClient：**
		- 用途：Nacos 的"命名服务"（服务注册与发现），用于服务注册/发现。
		- 典型操作：`RegisterInstance`(注册实例)、`DeregisterInstance`(注销实例)、`Subscribe/Unsubscribe`(订阅/取消订阅服务)、`SelectOneHealthyInstance`(选择健康实例)、`GetService`(获取服务)
	- **NewConfigClient：**
		- 用途：Nacos 的"配置管理"服务，用于加载/监听应用配置(如 YAML 文件)
		- 典型操作：`PublishConfig`(发布配置)、`GetConfig`(获取配置)、`ListenConfig`(监听配置变更)、`RemoveConfig`(删除配置)、`SearchConfig`(搜索配置)
	- 两者使用相同的 `constant.ClientConfig` 和 `[]constant.ServerConfig` 来配置连接信息（命名空间、认证、服务器地址、上下文路径 `/nacos`、协议等），但它们操作的是 Nacos 的不同模块，提供不同的方法。
-