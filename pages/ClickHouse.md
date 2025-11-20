## 启动容器
	- ```bash
	  docker run -d \
	    -p 18123:8123 \
	    -p 19000:9000 \
	    --name clickhouse-server-xiaoze-poc \
	    --ulimit nofile=262144:262144 \
	    -e CLICKHOUSE_USER=lyz \
	    -e CLICKHOUSE_PASSWORD=123456 \
	    clickhouse/clickhouse-server
	  ```
	- `8123:8123`：ClickHouse 的 HTTP 接口。
	- `9000:9000`：ClickHouse 原生 TCP 协议端口。给 `clickhouse-client` 等工具连接用。
	- `--ulimit nofile=262144:262144`：设置容器内的最大打开文件数限制。
		- 前一个 `262144`：软限制（进程当前能打开的最大文件数）。
		- 后一个 `262144`：硬限制（软限制的上限）。
		- ClickHouse 是高并发 OLAP 引擎，会打开大量文件和网络连接，官方建议把 `nofile` 调高，避免在压测或长时间运行时遇到 “too many open files” 之类错误。
	-
-