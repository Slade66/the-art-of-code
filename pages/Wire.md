概念
heading:: true
	- Wire 是 Google 开源的 Go 依赖注入（Dependency Injection，简称 DI）工具。
	- 它是一种编译时依赖注入工具，不在运行时注入依赖，也不是运行时框架，而是一个用于生成初始化代码的代码生成器。
	- 由于依赖注入原则（不是自己创建对象，而是从外部传入），外部对象必须通过构造函数传入。你需要手动创建对象，并按正确的顺序传入构造函数。如果需要注入的构造函数较多，就会有许多依赖需要手动组装，手动处理这些依赖会变得非常繁琐，编写的代码量也会大幅增加。这时，Wire 可以自动生成生成代码来注入依赖，简化这一过程。
	- 使用时，你需要先编写构造函数，通过构造函数来描述各个组件之间的依赖关系。Wire 会根据这些构造函数，静态分析每个函数的入参与返回值，构建出依赖的有向无环图（DAG），并反向推导出构造顺序，从最底层无依赖的对象开始，逐层向上生成初始化代码。
	- Wire 会根据你提供的构造函数，生成一个自动处理复杂依赖顺序的新构造函数。既避免了手动编写繁琐的构造代码，又不会带来运行时性能开销。
- 使用
  heading:: true
	- **安装 Wire：**
	  logseq.order-list-type:: number
		- `go install github.com/google/wire/cmd/wire@latest`
		- `go get github.com/google/wire`
	- **定义构造函数：**
	  logseq.order-list-type:: number
		- 在 Wire 中，依赖注入是通过构造函数来实现的。你需要为每个需要注入的对象定义一个构造函数，每个构造函数负责初始化并返回一个实例。
		  id:: 687349d6-6ec9-478b-8ed8-883982c56342
		- **示例：**
			- ```go
			  // Database 类型和构造函数
			  type Database struct {
			      Name string
			  }
			  
			  func NewDatabase() *Database {
			      return &Database{Name: "MySQL"}
			  }
			  
			  // Service 类型和构造函数
			  type Service struct {
			      DB *Database
			  }
			  
			  func NewService(db *Database) *Service {
			      return &Service{DB: db}
			  }
			  ```
	- **定义 ProviderSet：**
	  logseq.order-list-type:: number
		- 你可以通过 `wire.NewSet` 来创建一个 `ProviderSet`，将多个构造函数组合在一起。通过 `ProviderSet`，Wire 知道如何创建各个对象，并处理它们之间的依赖关系。
		- ```go
		  var MySet = wire.NewSet(NewDatabase, NewService)
		  ```
	- **创建用于生成注入器函数的初始化函数：**
	  logseq.order-list-type:: number
		- 定义一个函数，并调用 `wire.Build`。
		- ```go
		  func InitializeService() (*Service, error) {
		      wire.Build(MySet)
		      return nil, nil // 这行代码不会被执行，Wire 会自动生成代码来替代它
		  }
		  ```
	- **执行 `wire` 命令，生成注入器函数：**
	  logseq.order-list-type:: number
		- 在命令行执行 `wire` 命令，这时会生成一个 `wire_gen.go` 文件，里面包含了 `InitializeService` 的完整实现。
		- 它会类似这样：
			- ```go
			  func InitializeService() (*Service, error) {
			      db := NewDatabase()            // 创建 Database 实例
			      service := NewService(db)      // 创建 Service 实例，注入 Database
			      return service, nil            // 返回初始化完成的 Service 实例
			  }
			  ```
	- **使用生成的注入器函数：**
	  logseq.order-list-type:: number
		- Wire 会生成一个和 `InitializeService` 相似的函数，它会创建所有的依赖并返回初始化的对象。你可以在你的代码中直接调用这个生成的函数。
		- ```go
		  package main
		  
		  import (
		      "fmt"
		      "log"
		  )
		  
		  func main() {
		      // 调用 Wire 生成的注入器函数
		      service, err := InitializeService()
		      if err != nil {
		          log.Fatalf("Failed to initialize service: %v", err)
		      }
		  
		      // 使用注入的服务
		      fmt.Println(service.DB.Name)  // 输出: MySQL
		  }
		  ```
	- **构建标签：**
		- 用于控制文件的编译行为。
		- **这两行告诉 Go 编译器：**该文件只是用来给 Wire 工具生成代码，普通编译时会被忽略，不会包含在最终的可执行文件中。
		- `//go:build wireinject` 是 Go 1.17 及以后版本推荐使用的构建标签写法。
		- `// +build wireinject` 是较早版本的写法，用于兼容旧版本 Go。
		- 两者同时存在，确保在不同版本的 Go 环境下都能正确生效。
	- **`ProviderSet`：**
		- **作用：** `ProviderSet` 就是 Wire 中存放构造函数的容器，是构造函数的集合。通过 `ProviderSet`，你可以声明在初始化应用时，哪些依赖应该被注入到应用中，以及 Wire 如何创建依赖关系中的各个组件。然后，Wire 会根据这些声明自动为你生成依赖注入的代码。
		- **比喻：**
			- 想象你在组装一台复杂的机器，这台机器有很多零件，每个零件又依赖其他零件才能正常工作。
			- `ProviderSet` 就是一个“零件清单”，它列出了每个零件的制造过程（即构造函数）。
			- Wire 会根据这个清单，自动组合和装配这些零件，确保它们能够互相依赖、协同工作。
	- **`wire.NewSet`：**
		- **作用：**将多个“提供者”（即构造函数）组合成一个 `ProviderSet`，就是用来创建 `ProviderSet` 的工具。
		- 可以传入多个构造函数，也可以是现有的 `ProviderSet`，甚至是类型本身。
	- **注入器函数：**
		- 注入器函数是 Wire 生成的用于自动注入依赖的函数。它根据你提供的构造函数，自动创建对象并处理它们之间的依赖，最后返回一个初始化完成的对象。
		- 你将构造函数组织在一个 `ProviderSet` 中，然后定义一个初始化函数，在其中调用 `wire.Build` 并传入之前的 `ProviderSet`。接着，运行 `wire` 命令，Wire 会生成注入器函数，它会按顺序调用构造函数来创建实例，并将依赖项注入到需要它们的对象中，最终返回已初始化的对象。
-