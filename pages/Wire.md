概念
heading:: true
	- Wire 是 Google 开源的 Go 依赖注入（Dependency Injection，DI）工具。
	- **编译期依赖注入：**Wire 是一种编译时依赖注入工具，它不在运行时注入依赖，而是一个用于生成初始化代码的代码生成器，用生成代码替代反射，运行期零开销。
	- **问题：**
		- 由于依赖注入原则（不是自己创建对象，而是从外部传入），外部对象必须通过构造函数传入。
		- 你需要手动创建对象，并按正确的顺序传入构造函数。
		- 如果需要注入的构造函数较多，就会有许多依赖需要手动组装，手动处理这些依赖会变得非常繁琐，编写的代码量也会大幅增加。
		- 这时，Wire 可以自动生成生成代码来注入依赖，简化这一过程。
	- **核心原理：**使用时，你需要先编写构造函数，通过构造函数来描述各个组件之间的依赖关系。Wire 会根据这些构造函数，静态分析每个函数的入参与返回值，构建出依赖的有向无环图（DAG），并反向推导出构造顺序，从最底层无依赖的对象开始，逐层向上生成初始化代码，生成一个自动处理复杂依赖顺序的新构造函数，从而避免了手动编写繁琐的构造代码。
- 使用
  heading:: true
	- **安装 Wire：**
	  logseq.order-list-type:: number
		- `go install github.com/google/wire/cmd/wire@latest`
		- `go get github.com/google/wire`
	- **定义构造函数：**
	  logseq.order-list-type:: number
		- 你需要为每个需要注入的对象定义一个构造函数，每个构造函数负责初始化并返回一个实例。
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
		- 你可以通过 `wire.NewSet` 来创建一个 `ProviderSet`，将多个构造函数组合在一起。
		- ```go
		  var MySet = wire.NewSet(NewDatabase, NewService)
		  ```
	- **创建用于生成注入器函数的初始化函数：**
	  logseq.order-list-type:: number
		- 定义一个函数，并在其中调用 `wire.Build`。注意，该函数的返回值应为你期望生成的最终对象。
		- ```go
		  func InitializeService() (*Service, error) {
		      wire.Build(MySet)
		      return nil, nil // 这行代码不会被执行，Wire 会自动生成代码来替代它
		  }
		  ```
	- **执行 `wire` 命令，生成注入器函数：**
	  logseq.order-list-type:: number
		- 在命令行中执行 `wire` 命令后，会生成一个 `wire_gen.go` 文件，其中包含 `InitializeService` 的完整实现，用于创建所有依赖并返回初始化后的对象。
		- 它会像这样：
			- ```go
			  func InitializeService() (*Service, error) {
			      db := NewDatabase()            // 创建 Database 实例
			      service := NewService(db)      // 创建 Service 实例，注入 Database
			      return service, nil            // 返回初始化完成的 Service 实例
			  }
			  ```
	- **使用生成的注入器函数：**
	  logseq.order-list-type:: number
		- 你可以在代码中直接调用生成的最终构造函数。
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
	- `//go:build wireinject` 告诉 Go 编译器：该文件只是用来给 Wire 工具生成代码，在编译时会被忽略，不会包含在最终的可执行文件中。真正编译的是 `wire_gen.go`。
- **Provider：**构造函数。Wire 用它来构建对象。
- **ProviderSet：**
	- **作用：**ProviderSet 就是 Wire 中存放构造函数的容器，是构造函数的集合。
	- **比喻：**
		- 想象你在组装一台复杂的机器，这台机器有很多零件，每个零件又依赖其他零件才能正常工作。
		- `ProviderSet` 就是一个“零件清单”，它列出了每个零件的制造过程（即构造函数）。
		- Wire 会根据这个清单，自动组合和装配这些零件，确保它们能够互相依赖、协同工作。
- **Injector：**注入器。
	- 注入器函数是 Wire 生成的用于自动注入依赖的函数。
	- 首先，将构造函数组织到一个 `ProviderSet` 中；然后编写一个仅包含 `wire.Build(...)` 调用的函数，并在其中传入前面定义的 `ProviderSet`。接着，运行 `wire` 命令，Wire 会根据这些构造函数生成注入器函数的完整实现，按依赖关系顺序调用构造函数并注入依赖来创建对象，最终返回初始化完成的结果。
-