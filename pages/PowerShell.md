- `tree`
	- 显示当前目录及其所有子目录。
	- `/f`：显示每个目录中的文件名（不加 `/f` 只显示文件夹结构）。
- **如何设置环境变量？**
	- **永久生效：**
		- ```PowerShell
		  # 设置 HTTP 代理
		  [System.Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://127.0.0.1:10809", "User")
		  # 设置 HTTPS 代理
		  [System.Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://127.0.0.1:10809", "User")
		  ```
		- **注意：**执行完这些命令后，环境变量并不会立即在当前的 PowerShell 窗口中生效。你需要打开一个新的命令行窗口，才能看到永久化的设置。
- **如何查看当前会话的所有环境变量？**
	- ```PowerShell
	  Get-ChildItem Env:
	  ```
-