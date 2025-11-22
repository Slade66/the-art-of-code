-
- ## Docker
	- ```bash
	  docker run --name rbac-mysql -e MYSQL_ROOT_PASSWORD=123456 -v /mysql/data:/var/lib/mysql -p 3306:3306 -d mysql:8.4.7
	  ```
	- **环境变量：**
		- `MYSQL_ROOT_PASSWORD`：必填。指定 root 超级用户的密码。
		- `MYSQL_DATABASE`：可选。指定容器启动时要创建的数据库名字。
		- `MYSQL_USER` 和 `MYSQL_PASSWORD`：可选，但必须成对使用。用于创建一个新的用户并设置密码。这个用户会自动获得 `MYSQL_DATABASE` 指定数据库的全部权限。
			- **注意：**创建 root 用户不需要这两个变量，root 会自动创建，并使用你指定的 `MYSQL_ROOT_PASSWORD` 密码。
		- `MYSQL_ALLOW_EMPTY_PASSWORD`：可选。如果你把它设成一个非空值（比如 `yes`），就允许 root 用户用空密码启动。
		- `MYSQL_RANDOM_ROOT_PASSWORD`：可选。设成非空值（比如 `yes`），系统会自动生成一个 root 的随机密码。
			- 生成的密码会打印在容器启动日志里，长这样：`GENERATED ROOT PASSWORD: ...`
	- **存放数据：**
		- MySQL 默认会把数据写入 `/var/lib/mysql` 目录。
		- ```bash
		  mkdir -p /mysql/data
		  -v /mysql/data:/var/lib/mysql
		  ```
-