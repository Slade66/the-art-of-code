-
- #### Docker Desktop 的原理
	- Windows 本身并不运行 Docker Daemon，安装 Docker Desktop 时，它会自动创建并管理一个专用的 WSL 2 Linux 子系统，并将 Docker Daemon 运行其中。
	- ```powershell
	  PS C:\> wsl -l -v
	    NAME              STATE           VERSION
	  * Ubuntu            Running         2
	    docker-desktop    Running         2
	  ```
- [[WSL integration]]
-