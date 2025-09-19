- Go 语言规定：一个包（package）中只能有一个 `main` 函数。由于同一目录下的所有 `.go` 文件默认属于同一个包，因此不能在该目录的多个文件中同时定义 `main` 函数，否则会在编译时报错。
- 解决方案：创建不同的目录来自然地分离不同的 `main` 包。
	- **核心思想：**为每一个你想要生成的可执行文件，创建一个独立的子目录。通常，这些子目录都放在一个名为 `cmd` 的顶层文件夹下。
	- **目录结构示例：**
		- 假设你的项目 `my-project` 需要编译出两个程序：一个叫 `api-server`（API服务器），另一个叫 `migration-tool`（数据迁移工具）。
		- ```text
		  my-project/
		  ├── go.mod
		  ├── cmd/
		  │   ├── api-server/
		  │   │   └── main.go       // 这是 api-server 的入口 (package main)
		  │   └── migration-tool/
		  │       └── main.go       // 这是 migration-tool 的入口 (package main)
		  │
		  ├── internal/             // 项目内部共享的代码，外部无法导入
		  │   ├── database/
		  │   └── logger/
		  │
		  └── pkg/                  // 项目外部也可以导入的共享代码（如果有）
		      └── utils/
		  
		  ```
		- `cmd/api-server/main.go` 和 `cmd/migration-tool/main.go` 都声明自己是 `package main` 并包含 `func main()`。
		- 它们都可以自由地导入 `internal/` 或 `pkg/` 目录下的共享代码。
-