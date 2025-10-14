- **clickhouse-logger 插件有什么用？**
	- `clickhouse-logger` 插件用于将 API 请求的日志异步、批量地推送到 ClickHouse 数据库中。
- **clickhouse-logger 插件的工作原理**
	- **异步批量发送：**
		- 它不会让 API 请求等待日志写成功，而是先把日志放进一个队列里，然后每隔一段时间（比如 5 秒）或积攒到一定数量（比如 1000 条）再一次性发给 ClickHouse。
		- 这就像你点外卖，骑手不是送一单跑一趟，而是一次性取上好几单，顺路一起送，效率更高。
- **clickhouse-logger 插件的配置**
	- `endpoint_addrs`：ClickHouse 连接地址数组，支持多个地址。APISIX 会自动进行负载均衡和故障转移。
	- `database`：指定日志存储的 ClickHouse 数据库名称。
	- `logtable`：指定日志存储的表名。需提前在 ClickHouse 中创建该表，并确保表字段与日志格式匹配。
	- `user`：ClickHouse 的用户名。
	- `password`：ClickHouse 的密码。这个字段会自动加密后存储在 etcd 中。
	- `timeout`：连接超时时间，表示 APISIX 发送日志到 ClickHouse 后，连接保持多长时间。若超出该时间没有新操作，连接会自动关闭。默认值为 3 秒。
	- `log_format`：自定义日志格式，允许你定制日志的内容和结构。
	  id:: 68ee0a0b-8bf1-4514-a55d-cd4541dad5ac
	- `include_req_body`：是否记录请求体。当设为 `true` 时，APISIX 会记录客户端 POST/PUT 请求的 Body 内容。
	- `include_req_body_expr`：只有符合条件的请求 Body 会被记录。
	- `include_resp_body`：是否记录响应体，记录 API 返回给客户端的 Body 内容。
	- `include_resp_body_expr`：当此条件为真时，响应的 Body 才会被记录到日志中。
- **clickhouse-logger 插件的元数据配置**
	- ((68ee0a0b-8bf1-4514-a55d-cd4541dad5ac))
		- 日志格式由一组键值对（JSON 格式）表示。值只能是字符串类型。你可以使用 APISIX 或 Nginx 的变量，方法是在变量名前加上 `$`。例如，`$host` 表示请求的主机名，`$time_iso8601` 表示 ISO 8601 格式的时间，`$remote_addr` 表示客户端 IP 地址。
		- 配置插件的元数据是全局生效的，意味着它会应用到所有使用 `clickhouse-logger` 插件的路由和服务。你设置的 `log_format` 会在这些路由和服务中被使用。
		- **示例：**
			- ```bash
			  curl http://127.0.0.1:9180/apisix/admin/plugin_metadata/clickhouse-logger \
			  -H "X-API-KEY: $admin_key" \
			  -X PUT -d '
			  {
			      "log_format": {
			          "host": "$host",
			          "@timestamp": "$time_iso8601",
			          "client_ip": "$remote_addr"
			      }
			  }'
			  ```
			- 这个请求会将日志格式配置为：记录主机名（`$host`）、时间戳（`$time_iso8601`）和客户端 IP 地址（`$remote_addr`）。
-