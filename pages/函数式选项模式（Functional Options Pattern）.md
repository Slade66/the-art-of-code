## 函数式选项模式是什么？
	- **核心思想：**
		- 函数式选项模式利用了 Go 语言中函数是“一等公民”的特性，通过传入一系列函数来配置和初始化目标对象。这种方式能够在不改变函数签名的情况下，以一种优雅且可读性高的方式，增加任意多的可选配置。
	- **代码示例：**
		- ```go
		  package main
		  
		  import (
		  	"fmt"
		  	"time"
		  )
		  
		  // Server 是一个需要配置的结构体
		  type Server struct {
		  	Host    string
		  	Port    int
		  	Timeout time.Duration
		  }
		  
		  // Option 是一个函数类型，用于修改 Server 配置
		  type Option func(s *Server)
		  
		  // WithHost 是一个选项函数构造器，用于配置 Host
		  func WithHost(host string) Option {
		  	return func(s *Server) {
		  		s.Host = host
		  	}
		  }
		  
		  // WithPort 是一个选项函数构造器，用于配置 Port
		  func WithPort(port int) Option {
		  	return func(s *Server) {
		  		s.Port = port
		  	}
		  }
		  
		  // WithTimeout 是一个选项函数构造器，用于配置 Timeout
		  func WithTimeout(timeout time.Duration) Option {
		  	return func(s *Server) {
		  		s.Timeout = timeout
		  	}
		  }
		  
		  // NewServer 是构造函数，接收可变参数的 Option 函数
		  func NewServer(opts ...Option) *Server {
		  	// 1. 设置默认值
		  	s := &Server{
		  		Host:    "0.0.0.0",
		  		Port:    8080,
		  		Timeout: 30 * time.Second,
		  	}
		  
		  	// 2. 遍历选项函数并应用配置
		  	for _, opt := range opts {
		  		opt(s)
		  	}
		  
		  	return s
		  }
		  
		  func main() {
		  	// 示例 1: 使用默认配置
		  	server1 := NewServer()
		  	fmt.Printf("Server 1: %+v\n", server1)
		  	// 输出: Server 1: {Host:0.0.0.0 Port:8080 Timeout:30s}
		  
		  	// 示例 2: 自定义端口
		  	server2 := NewServer(WithPort(9000))
		  	fmt.Printf("Server 2: %+v\n", server2)
		  	// 输出: Server 2: {Host:0.0.0.0 Port:9000 Timeout:30s}
		  
		  	// 示例 3: 自定义多个配置
		  	server3 := NewServer(
		  		WithHost("localhost"),
		  		WithTimeout(60*time.Second),
		  	)
		  	fmt.Printf("Server 3: %+v\n", server3)
		  	// 输出: Server 3: {Host:localhost Port:8080 Timeout:1m0s}
		  
		  	// 示例 4: 自定义所有配置
		  	server4 := NewServer(
		  		WithHost("127.0.0.1"),
		  		WithPort(9090),
		  		WithTimeout(10*time.Second),
		  	)
		  	fmt.Printf("Server 4: %+v\n", server4)
		  	// 输出: Server 4: {Host:127.0.0.1 Port:9090 Timeout:10s}
		  }
		  ```
- ## 函数式选项模式解决了什么问题？
	- ### 不使用函数式选项模式的问题
		- 我们常常需要创建复杂对象。如果不使用函数式选项模式，常见的构造方式主要有以下几种，但都存在一定的缺陷：
		- **单构造函数多参数：**
			- ```go
			  // 所有配置都作为函数参数传入
			  func NewServer(host string, port int, timeout time.Duration, maxConns int) *Server {
			      // ... 实现
			  }
			  ```
			- **缺点：**
				- **可读性差：**当参数过多时，函数签名会变得很长，不仅可读性差，也难以直观理解每个参数的含义。
				- **调用繁琐：**对于可选参数，调用者仍然需要传入零值来占位。
				- **修改成本高：**如果未来需要增加新的可选参数，必须修改函数签名，所有调用该函数的地方都需要跟着修改，这会带来巨大的修改成本。
		- **多构造函数：**
			- ```go
			  // 为不同的参数组合提供不同的构造函数
			  func NewServer(host string, port int) *Server {
			      // ...
			  }
			  
			  func NewServerWithTimeout(host string, port int, timeout time.Duration) *Server {
			      // ...
			  }
			  
			  func NewServerWithTimeoutAndMaxConns(host string, port int, timeout time.Duration, maxConns int) *Server {
			      // ...
			  }
			  ```
			- **缺点：**
				- **构造函数过多：**随着配置项的增多，需要编写大量仅存在细微差异的构造函数。这些函数本质上是重复代码，维护起来非常困难。
		- **配置结构体：**
			- ```go
			  type Config struct {
			      Host string
			      Port int
			  }
			  
			  func NewServer(config Config) {
			      port := 8080 // 默认值
			      if config.Port != 0 {
			          port = config.Port // 用户设置了非零值，就用它
			      }
			      // ...
			  }
			  ```
			- **缺点：**
				- **零值模糊性：**
					- 当某个字段的值恰好是其类型的零值时（如 `int` 的 `0`、`string` 的 `""`），程序就无法区分这两种情况：一种是用户未设置，希望采用默认值；另一种是用户明确地想要设置为零值。
					- 因此，如果用户的真实意图就是设置零值，代码也会错误地判断为“未设置”，并强制应用默认值，这便违背了用户的真实意图。
					- 结构体传参的最大问题在于，它无法清晰区分“缺失”和“零值”这两种不同的意图，迫使你借助指针或额外字段来弥补，最终让 API 变得不够直观和易用。
		- **Setter 方法**：
			- ```go
			  // 先创建一个基本对象，然后通过 Setter 方法进行配置
			  func NewServer(host string, port int) *Server {
			      // ... 创建一个有默认值的 server
			  }
			  
			  func (s *Server) SetTimeout(timeout time.Duration) {
			      s.timeout = timeout
			  }
			  
			  func (s *Server) SetMaxConns(maxConns int) {
			      s.maxConns = maxConns
			  }
			  
			  // 调用方式
			  server := NewServer("localhost", 8080)
			  server.SetTimeout(30 * time.Second)
			  server.SetMaxConns(100)
			  ```
			- **缺点：**
				- **配置步骤过多：**对象创建后无法立即使用，需要通过额外调用多个 Setter 方法来完成配置。
				- **非原子操作：**对象的配置过程被分成了多个步骤，并非“原子操作”。这意味着在调用完所有 Setter 之前，对象处于一个不完整的状态。
	- ### 函数式选项模式解决了以下问题
		- **参数过多**：避免了构造函数参数过多导致函数签名过长、难以阅读和使用的问题。
		- **扩展性差**：在增加新的配置项时，无需修改已有函数签名，保证了向后兼容性。
		- **可读性差**：通过具名选项的方式，使每个配置项的意图一目了然，解决了参数含义不清晰的问题。
		- **零值困扰**：能够清晰区分用户是想设置零值，还是没有设置参数，避免了默认值与零值意图混淆的问题。
- ## 函数式选项模式的组成角色与执行流程
	- ### 组成角色
		- **目标对象：**这是选项模式最终要创建和配置的核心结构体。所有后续的配置操作，都是为了修改和填充这个结构体的字段。
		- **选项函数类型：**定义了所有配置函数的统一签名，通常是一个函数类型。该函数接收一个指向目标对象的指针，并对其进行修改。这确保了所有配置函数都具有相同的入参和修改逻辑，为构造函数提供了统一的参数规范。
		- **选项函数生成器：**这些函数是构建具体配置的工厂。它们就像一个“配置加工车间”，接收原始的配置参数（比如一个超时时间），然后按照统一的“选项函数类型”标准，生产出一个可以直接作用于目标对象的配置函数。
		  id:: 68b6ff7c-cf46-45d7-a1d2-e697db0f0411
		- **构造函数：**这是创建目标对象的入口。它接收一个或多个“选项函数”作为可变参数。在函数内部，它会首先创建一个带有默认值的目标对象，然后依次执行所有传入的选项函数，通过它们来修改和覆盖默认配置，最终返回一个配置完成的对象。
	- ### 执行流程
		- **调用构造函数**：客户端首先调用构造函数。
		  logseq.order-list-type:: number
		- **传入配置函数**：调用时，客户端会传入构造函数所需的参数，并通过一系列选项函数生成器来构建具体的配置函数，然后将这些函数作为可变参数传入。
		  logseq.order-list-type:: number
		- **创建默认对象**：构造函数在内部会先创建一个带有默认值的目标对象。
		  logseq.order-list-type:: number
		- **遍历并应用配置**：接着，构造函数会遍历所有传入的配置函数。
		  logseq.order-list-type:: number
		- **修改对象状态**：在遍历过程中，它会依次执行每个配置函数，将默认对象作为参数传入。每个配置函数都根据自身的逻辑，修改目标对象的内部字段。
		  logseq.order-list-type:: number
		- **返回最终对象**：当所有配置函数执行完毕后，构造函数返回这个经过完整配置的目标对象。
		  logseq.order-list-type:: number
- ## 函数式选项模式的优缺点
	- ### 优点
		- **可读性强：**
			- 函数式选项模式通过使用命名清晰的函数来配置参数，使得调用代码如同阅读自然语言。这种方式直观且自解释，例如，`NewServer(WithTimeout(...), WithLogger(...))` 清晰地表明了正在设置服务器的超时时间和日志记录器，大大提高了代码的可读性。
		- **向后兼容性强：**
			- 当你需要为结构体添加新配置时，只需新增一个选项函数，而无需修改构造函数的代码。由于构造函数采用可变参数 `...Option`，它天然支持扩展，让你能在不影响现有代码的情况下轻松添加新功能。
		- **高度可组合：**
			- 选项函数是独立的、可重用的代码块，你可以将多个选项组合起来，并将其封装成一个更高级的函数，或者存储在切片等数据结构中。这种方式能让你轻松地在不同场景下复用配置，从而简化复杂的设置。
			- 例如，你可以创建一个 `WithDefaultSecurity()` 函数，其中包含了 `WithAuthToken()` 和 `WithTLS()` 两个选项，这样就能在多个服务中一键应用默认的安全配置。
	- ### 缺点
		- **增加了样板代码：**
			- 该模式最大的缺点是会产生较多的样板代码。你需要为每一个可配置的字段定义一个独立的选项函数。当一个结构体有几十个配置项时，这种模式会显著增加代码量，使得文件变得庞大。对于只有一两个配置项的简单结构体，这种模式可能会显得过于复杂，直接使用一个简单的构造函数或配置结构体可能更为简洁。
- ## 函数式选项模式的适用场景
	- ### 适用场景
		- **多个可选参数：**当一个对象的构造函数需要处理多个可选参数时，此模式是理想选择。它能避免构造函数参数列表的爆炸式增长，使代码更整洁。
		- **API 向后兼容性：**如果你需要保证 API 的长期稳定性，该模式能让你在未来轻松增加新配置项，而不会破坏现有用户的代码。
		- **代码意图清晰：**当你希望调用方的代码能够清晰、自解释地表达其配置意图时，此模式能通过命名直观的选项函数来实现。
	- ### 不适用场景
		- **无可选参数：**如果一个对象的所有配置参数都是必需的，那么使用该模式是一种过度设计。
		- **配置项少：**对于只有少量配置项的简单结构体，使用该模式会显得过于复杂，此时一个简单的构造函数或配置结构体能更简洁地完成任务。
- ## 函数式选项模式与其它相似模式的比较
	- **函数式选项模式 vs. 建造者模式：**
		- **建造者模式：**
			- 建造者模式（Builder Pattern）是另一种常用于处理复杂对象创建的模式，在 Java、C# 等面向对象语言中非常流行。在 Go 语言中，它的实现通常是这样的：
				- ```go
				  // Go 风格的 Builder
				  b := NewServerBuilder("localhost", 8080)
				  srv := b.WithTimeout(5 * time.Second).WithMaxConns(200).Build()
				  ```
		- **不同之处：**
			- **状态：**构建器模式会创建一个临时的 Builder 对象来保存中间配置状态，最后通过 `Build()` 方法生成最终对象。而函数式选项模式是无状态的，它通过函数闭包直接操作目标对象。
			- **语法：**构建器模式采用链式调用的语法（`obj.Method1().Method2()`），而函数式选项模式则使用函数作为参数的语法（`New(Opt1(), Opt2())`）。
			- **地道性：**在 Go 语言中，函数式选项模式通常被认为是更地道、更符合 Go 风格的设计。而构建器模式在 Go 中也可用，但通常更受那些从其他面向对象语言转来的开发者青睐。
			- **可组合性：**函数式选项模式的选项函数是独立的，可以存到变量或切片里，更容易进行组合和复用。而构建器模式的链式调用语法使得这种灵活组合相对困难。
-