问题复现
heading:: true
	- 假设你使用以下命令启动了一个 MySQL 容器：
	- ```bash
	  docker run -d --name mysql-demo -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql
	  ```
	- 然后尝试在 WSL 或终端中使用 `mysql` 命令连接：
	- ```bash
	  mysql -u root -p
	  ```
	- 结果出现类似如下报错：
	- ```text
	  Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'
	  ```
- 问题分析
  heading:: true
	- MySQL 客户端与服务器通信有两种方式可选：
		- 通过 TCP/IP 网络连接（如 `127.0.0.1:3306`）
		  logseq.order-list-type:: number
		- 通过本地 socket 文件连接（UNIX 域套接字）
		  logseq.order-list-type:: number
	- 默认情况下，MySQL 客户端会使用本地 socket 文件连接，路径为：`/var/run/mysqld/mysqld.sock`。
	- Socket 文件是一种本地进程通信机制（IPC），是 Linux/Unix 系统中的特殊文件类型，允许两个本地程序通过文件系统进行通信，而无需使用网络端口。
	- 然而，当 MySQL 实际运行在 Docker 容器中，系统中并不存在这个 socket 文件，因此连接失败。
- 解决方案
  heading:: true
	- 通过显式指定 TCP 网络连接方式，连接容器中的 MySQL：
	- ```bash
	  mysql -h 127.0.0.1 -P 3306 -u root -p
	  ```
	- `-h 127.0.0.1`：强制使用 TCP 连接，而不是默认的 socket 方式。
-