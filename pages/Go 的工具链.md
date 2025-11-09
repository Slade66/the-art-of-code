- 工具链指的是一系列工具，它们相互配合，共同完成从编写代码到最终生成可执行文件的完整流程。
- `go run`
  collapsed:: true
	- **作用**：快速编译并立刻执行 Go 程序。
	- **流程**：
	  collapsed:: true
		- 在后台先调用 `go build`，生成一个临时的可执行文件（存在临时目录）。
		- 立刻执行这个二进制文件。
		- 执行完毕后，临时的二进制文件会被自动删除。
	- **例子**：
	  collapsed:: true
		- ```bash
		  go run main.go
		  ```
	- **适用于**：不关心生成的可执行文件，只要运行结果。
- `go build`
  collapsed:: true
	- **作用**：把 Go 源代码编译成可执行文件，但不自动运行。
	- **适用于**：
	  collapsed:: true
		- 你想保存这个可执行文件（不只是跑一次而已）。
		- 可执行文件可以直接分发给用户执行，即使用户的系统中没有安装 Go 环境。
- `go get 包路径`
  collapsed:: true
	- **作用**：用于拉取依赖包，并将它们写入到 `go.mod` 文件中。
	- **原理：**
		- `go get` 命令会根据提供的包路径，自动使用 Git 拉取源代码，并下载到本地的模块缓存目录（`$GOPATH/pkg/mod/`）。同时，`go.mod` 和 `go.sum` 文件也会更新，记录相应的模块版本及其校验信息。
	- **常见原因和解决办法**：
		- **网络问题**：
			- `go get` 需要从远程仓库下载 Go 依赖包源码及其依赖，而这些仓库通常托管在 GitHub 等国外服务器上，访问速度较慢或被墙。而且 Go 命令行工具默认不会使用系统代理（而浏览器通常会使用），因此如果不手动配置代理环境变量或使用国内 Go 代理，就会导致 `go get` 下载失败或超时。
			- **配置国内 Go 模块代理**：`go env -w GOPROXY=https://goproxy.cn,direct`
		- **在非模块文件夹下使用**：
			- `go get` 只能在 Go module 项目下使用（需要 `go.mod` 文件），否则会报错。
			- 如果在空目录中使用，需先执行 `go mod init` 初始化，再用 `go get` 添加依赖。
- `go test`
  collapsed:: true
	- **作用**：自动化执行当前目录下所有以 `_test.go` 结尾的测试文件。
- `go mod`
  collapsed:: true
	- `go mod init 模块名`
	  collapsed:: true
		- 模块名即模块路径，也就是源代码所在的仓库位置。如果你打算将模块发布供他人使用，模块路径必须是 Go 工具能够下载的地址。
		- **作用：**
		  collapsed:: true
			- 用于启动 Go 项目的模块化管理，确保项目能正确跟踪和管理外部依赖。
			- 该命令将在当前目录创建一个 `go.mod` 文件，标识当前目录为 Go 模块。文件中记录了模块名称、Go 版本以及所有依赖的外部库和其版本信息，用于管理项目中的第三方依赖及版本。
			- 同时，命令会生成一个 `go.sum` 文件，用于记录依赖模块的哈希值（校验和）。校验和确保下载的依赖包完整、安全，防止被篡改或损坏，从而保证依赖模块在不同环境中的一致性和安全性。
	- `go mod tidy`
	  collapsed:: true
		- **作用：**用于清理和优化 `go.mod` 和 `go.sum` 文件。它会自动删除未使用的依赖，并将代码中使用但未声明的依赖添加到 `go.mod` 中，同时自动下载缺失的依赖。
	- `go mod download`
	  collapsed:: true
		- **作用：**用于下载当前项目所依赖的所有模块及其版本，并将它们存储在本地 Go 模块缓存目录（`$GOPATH/pkg/mod`）中。之后，模块可以直接从本地缓存加载，避免在后续构建或运行时重复从网络下载，从而减少构建时间，并支持离线构建。
- `go fmt`
  collapsed:: true
	- **作用**：格式化 Go 源代码，使其符合官方的代码风格。
- `go install`
  collapsed:: true
	- **作用**：去 GitHub 上下载项目的源代码，然后用本地 Go 编译器将源代码编译成二进制文件，并将其安装到 `$GOPATH/bin`，让你在任何地方都能运行它。
- `go doc`
  collapsed:: true
	- **作用**：在命令行中查看包、函数、类型、接口和变量等的文档注释。
- `go env`
  collapsed:: true
	- **作用：**输出当前 Go 环境的所有配置信息。
	- **环境变量：**
	  collapsed:: true
		- `GO111MODULE`
		  collapsed:: true
			- **作用：**决定是否启用 Go 模块。
			- **off**：禁用 Go Modules，使用传统的 `$GOPATH` 模式。
			- **on**：始终启用 Go Modules，无论是否位于 `$GOPATH` 下。
			- **auto**：当 `GO111MODULE` 未设置具体值时，Go 会默认使用 `auto` 模式。如果当前目录中存在 `go.mod` 文件，则启用 Go Modules；否则，使用传统的 `$GOPATH` 模式。
		- `GOPROXY`
		  collapsed:: true
			- **作用：**决定 Go 通过哪个代理获取模块。
- `go generate`
  collapsed:: true
	- **作用：**`go generate` 是一个代码生成工具，它会扫描你项目中的 Go 源文件（`.go` 文件），找出并执行一种特殊的注释 `//go:generate` 后面的命令。这些命令通常用于在编译前自动生成 Go 代码，从而避免手动编写大量重复、有规律却非常枯燥的样板代码，极大地提高了开发效率。
	- **使用 `//go:generate`：**
	  collapsed:: true
		- 当你希望 `go generate` 执行某个命令时，你需要在任意一个 `.go` 文件中，以单行注释的形式写入这个命令。
		- **语法：**`//go:generate command argument1 argument2 ...`
		- **规则：**
		  collapsed:: true
			- 必须以 `//` 开头，`//` 和 `go:generate` 之间不能有空格。
			- `go:generate` 后面跟着的就是要执行的命令，和你在终端里输入的基本一样。
			- 这个注释可以放在文件的任何位置，但通常放在与它相关的代码附近，以提高可读性。
			- 一个文件可以包含多条 `//go:generate` 指令，`go generate` 会按照它们在文件中出现的顺序依次执行。
	- **示例：**
	  collapsed:: true
		- `go generate ./...`：从当前目录开始，递归地扫描所有子目录下的 Go 源文件，打开所有 `.go` 文件，逐行查找 `//go:generate` 注释，并执行它们。
		  collapsed:: true
			- `./...`：这是一个在 Go 工具链中非常常见的路径通配符，意思是“从当前目录（`.`）开始，递归地（`...`）查找所有子目录”。
	- **泽话总结：**`//generate 命令 参数` 类似于注解，当执行 `go generate` 命令时，这行写在注释中的命令会被自动执行。
- `go help`
  collapsed:: true
	- **作用：**用于查看 Go 命令及其子命令的文档，快速了解命令的使用方法、选项及相关说明。
	- **用法：**
	  collapsed:: true
		- `go help`：列出所有可用的 Go 命令以及简短的描述。
		- `go help <command>`：查看某个命令的详细帮助信息。
		- `go help <command> <subcommand>`：查看某个子命令的帮助信息。
- **创建一个 Go 的新项目：**
  collapsed:: true
	- ```bash
	  go version
	  mkdir myproject && cd myproject
	  go mod init github.com/你的用户名/myproject
	  ```
- **多文件编译机制：**
  collapsed:: true
	- 当代码分布在多个文件中时，这些文件必须同时参与编译。否则，编译器在检查第一个文件时会发现某个类型未在该文件中定义，而由于未指定包含其定义的其他文件，编译器无法找到对应声明，从而报出 **`undefined`** 错误。
	- 因此，在编译或运行由多个文件组成的 Go 程序时，应**同时将所有源文件传递给 `go run`**。
-