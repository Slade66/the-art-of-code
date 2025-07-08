概念
heading:: true
	- Wire 是 Google 开源的 Go 依赖注入（Dependency Injection，简称 DI）工具。
	- 它是一种编译时依赖注入工具，不在运行时注入依赖，也不是运行时框架，而是一个用于生成初始化代码的代码生成器。
	- 使用时，你需要先编写构造函数，通过构造函数来描述各个组件之间的依赖关系。Wire 会根据这些构造函数，静态分析每个函数的入参与返回值，构建出依赖的有向无环图（DAG），并反向推导出构造顺序，从最底层无依赖的对象开始，逐层向上生成初始化代码。
	- Wire 会根据你提供的构造函数，生成一个自动处理复杂依赖顺序的新构造函数。既避免了手动编写繁琐的构造代码，又不会带来运行时性能开销。
- 使用
  heading:: true
	- **安装 Wire：**
		- `go install github.com/google/wire/cmd/wire@latest`
		- `go get github.com/google/wire`
	- **示例代码：**
		- ```go
		  // file: main.go
		  package main
		  
		  import (
		  	"fmt"  // 导入格式化输出包
		  )
		  
		  // 定义结构体 Foo，包含一个字符串字段 Name
		  type Foo struct {
		  	Name string
		  }
		  
		  // NewFoo 是 Foo 的构造函数，返回指向 Foo 实例的指针
		  // 这里初始化 Name 字段为 "foo"
		  func NewFoo() *Foo {
		  	return &Foo{Name: "foo"}
		  }
		  
		  // 定义结构体 Bar，包含一个指向 Foo 的指针字段 Foo
		  // 表示 Bar 依赖 Foo
		  type Bar struct {
		  	Foo *Foo
		  }
		  
		  // NewBar 是 Bar 的构造函数，接收一个 Foo 指针作为依赖参数
		  // 返回初始化后的 Bar 实例指针
		  func NewBar(foo *Foo) *Bar {
		  	return &Bar{Foo: foo}
		  }
		  
		  // main 函数是程序的入口点
		  func main() {
		  	// 调用 InitializeBar() 获取一个完整初始化的 Bar 实例
		  	// InitializeBar 是由 Wire 自动生成的函数，
		  	// 负责调用 NewFoo 和 NewBar 并组装依赖关系
		  	bar := InitializeBar()
		  
		  	// 输出 Bar 中 Foo 的 Name 字段值
		  	fmt.Println("Bar contains Foo with name:", bar.Foo.Name)
		  }
		  ```
		- ```go
		  // file: wire.go
		  //go:build wireinject
		  // +build wireinject
		  
		  package main
		  
		  import "github.com/google/wire"
		  
		  // 声明一个 provider set，包含所有构造函数
		  // Wire 会根据这里的构造函数，自动构建依赖关系图
		  var fooSet = wire.NewSet(NewFoo, NewBar)
		  
		  // injector 函数，负责初始化并返回一个完整的 Bar 实例
		  // 这个函数的具体实现由 Wire 自动生成
		  func InitializeBar() *Bar {
		  	// wire.Build 告诉 Wire 使用 fooSet 中的构造函数进行依赖注入
		  	wire.Build(fooSet)
		  
		  	// 这行返回语句不会被编译进最终可执行文件，
		  	// Wire 会生成新的实现代码替换这里
		  	return &Bar{}
		  }
		  ```
		- 然后在命令行执行 `wire`，这时会生成一个 `wire_gen.go` 文件，里面包含了 `InitializeBar` 的完整实现。
		- **构建标签：**
			- 用于控制文件的编译行为。
			- **这两行告诉 Go 编译器：**该文件仅在运行 Wire 工具生成代码时生效，普通编译时会被忽略，不会包含在最终的可执行文件中。
			- `//go:build wireinject` 是 Go 1.17 及以后版本推荐使用的构建标签写法。
			- `// +build wireinject` 是较早版本的写法，用于兼容旧版本 Go。
			- 两者同时存在，确保在不同版本的 Go 环境下都能正确生效。
-