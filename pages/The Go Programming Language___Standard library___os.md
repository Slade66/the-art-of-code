- **`os` 包有什么用？**
  collapsed:: true
	- `os` 包是 Go 标准库中用于平台无关地与操作系统进行交互的核心包。
	- 无论底层是 Linux、Windows 还是 macOS，都使用相同的 Go 函数名，源代码无需修改即可在不同系统上运行。运行时通过交叉编译机制，根据 `GOOS`（Operating System）和 `GOARCH`（Architecture）环境变量，自动选择对应平台的底层系统调用实现来链接，生成正确的二进制文件。
	- 如果 `os` 包不存在，您可能需要直接使用 Go 的 `syscall` 包，那时您就必须编写特定于平台的代码，例如：
		- ```go
		  # 编译一个可以在 Linux/AMD64 机器上运行的程序
		  GOOS=linux GOARCH=amd64 go build
		  
		  # 编译一个可以在 Windows/AMD64 机器上运行的程序
		  GOOS=windows GOARCH=amd64 go build
		  
		  // 伪代码：在不同系统上，实现可能需要不同逻辑
		  if runtime.GOOS == "windows" {
		      // 调用 Windows API 的逻辑
		  } else {
		      // 调用 Unix/Linux 系统调用的逻辑
		  }
		  ```
		- Go 的 `os` 包就是将这种复杂的条件判断和系统调用封装起来，为您提供了这个“统一的、平台无关的接口”。
- **`os` 包函数的类别划分：**
  collapsed:: true
	- **文件系统和 I/O 操作：**
		- **读写快捷函数：** `ReadFile`, `WriteFile`。
		- **文件打开/创建：** `Open`, `Create`, `OpenFile`, `CreateTemp`, `NewFile`。
		- **文件/目录管理：** `Remove`, `RemoveAll`, `Rename`, `Link`, `Symlink`, `Mkdir`, `MkdirAll`, `MkdirTemp`。
		- **文件信息/权限：** `Stat`, `Lstat`, `Chmod`, `Chown`, `Chtimes`, `Truncate`。
		- **目录读取：** `ReadDir`。
- `func ReadFile(name string) ([]byte, error)`
  collapsed:: true
	- 读取指定名称的文件，并返回该文件的全部内容。
	- **注意：**
		- **内存占用：**`os.ReadFile` 会将文件的全部内容加载到内存中的一个字节切片里。如果文件非常大（例如几个 GB），这可能会导致内存不足（OOM）或其他性能问题。
- `func WriteFile(name string, data []byte, perm FileMode) error`
  collapsed:: true
	- **作用：**
		- `WriteFile` 会将数据写入指定名称的文件，并在需要时自动创建该文件。
		- 如果文件不存在，`WriteFile` 会使用参数 `perm` 指定的权限创建它；
		- 如果文件已存在，则会在写入前**清空文件内容**，但不会修改其原有权限。
	- **参数：**
		- `perm FileMode`：
			- 表示文件权限，用于指定新建文件的访问权限。
			- `FileMode` 的值是一个**八进制数**（前面要写 `0`）。
			- 它的三位分别代表 **文件所有者、同组用户、其他用户** 的权限。
			- 权限值及其含义：
				- 4，读
				- 2，写
				- 1，执行
			- 组合方式：把需要的权限值相加即可。
-