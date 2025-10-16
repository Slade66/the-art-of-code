- **方法一：在 WSL 中访问 Windows 的 C 盘**
	- WSL 会自动挂载 Windows 的磁盘，你可以通过路径 `/mnt/c/` 直接访问 C 盘的内容。
- **方法二：在 Windows 资源管理器中访问 WSL 文件系统**
	- 如果使用的是 WSL 2，可以在 Windows 资源管理器的地址栏中输入以下路径，直接访问 WSL 的文件系统：
	- ```powershell
	  \\wsl$\Ubuntu\
	  ```
	- 也可以通过资源管理器左侧侧边栏的 Linux 入口快速访问。
-