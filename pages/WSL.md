WSL 是什么？
heading:: true
	- WSL（Windows Subsystem for Linux，Windows 子系统 Linux）是微软推出的一项功能，允许你在 Windows 上以轻量方式运行原生的 Linux 环境，无需安装虚拟机或配置双系统。
	- WSL 2 使用的是由微软官方维护的真实 Linux 内核。该内核基于开源版本（如 Linux 5.15 LTS），经过裁剪和适配（包括移除冗余驱动、添加与 Windows 通信的补丁）后，由微软自行编译并发布，专为在 Windows 上的 WSL 2 环境运行而设计。
- [[Windows 与 WSL 文件互传]]
- ## 安装 WSL
	- **访问：**https://learn.microsoft.com/en-us/windows/wsl/install#offline-install
	- **访问：**https://github.com/microsoft/wsl/releases
	- 下载 .msi 文件，然后点击执行。
- ## WSL 的常用命令
	- `wsl`：启动默认的 Linux 发行版。
	- `wsl --install <Distribution Name>`：安装指定的 Linux 发行版。
	- `wsl --list --online`：列出可供安装的 Linux 发行版。
	- `wsl --list --verbose`：查看所有已安装的 Linux 发行版、它们的状态以及运行的 WSL 版本（WSL 1 或 WSL 2）。
	- `wsl --unregister <Distribution Name>`：注销并永久删除指定的发行版的所有数据、设置和软件。
-