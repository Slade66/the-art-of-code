- **Upstream 是什么？**
	- 上游（Upstream）是请求的接收方和处理方，也就是你的各种后端服务。
	- 比如，你有三台服务器，它们都部署了完全相同的用户服务。那么，这三台服务器节点就可以组成一个 `Upstream`，`Upstream` 就是你的后端服务集群。
	- Upstream 就是对你的后端服务集群的一个逻辑上的定义，它告诉 APISIX：“嘿，这里有几个服务器都能处理某一类请求，你按照我说的规则，把请求分发给它们就行。”
- **为什么叫 “Upstream”？**
	- 你可以想象请求像水流一样，从客户端（下游）流向网关，再由网关流向（上游的）后端服务。
- **Upstream 有什么用？**
	- **执行负载均衡：**当一个 `Upstream` 中有多个节点时，APISIX 会根据配置的负载均衡规则，将请求分发到各节点，从而避免单个节点过载。
- **Upstream API**
	- **Upstream 资源请求路径：**`/apisix/admin/upstreams/{id}`
	- **请求体参数：**
		- Upstream 配置定义了如何对一组服务节点进行负载均衡、健康检查、重试以及连接管理。
		- `type`：负载均衡算法。
		  collapsed:: true
			- `roundrobin`（加权轮询）：默认值。APISIX 会像发牌一样，把请求依次发给节点列表中的每一个服务器。通过设置不同的权重，可以让性能更好的服务器处理更多的请求。
			- `chash`（一致性哈希）：它解决了 “会话保持” 问题。
				- **问题场景：**用户登录后，会话信息存储在服务器 A 上。如果下一次请求被 `roundrobin` 分配到服务器 B，就会导致用户需要重新登录。
				- **解决方案：**`chash` 会根据客户端的某个固定特征（如 IP 地址、请求头中的 `user-id`，或某个 `cookie`）计算哈希值，并确保同一特征的请求始终转发到同一台后端服务器。只要该服务器正常运行，用户的请求就会持续由它处理。
				- **相关参数**：使用 `chash` 时，你需要配合 `hash_on` 和 `key` 参数来指定到底根据什么信息来做哈希。
		- `nodes`：后端服务节点列表。
		  collapsed:: true
			- **哈希表格式：**
				- ```json
				  "nodes": {
				    "192.168.1.100:80": 1, 
				    "httpbin.org:443": 2,
				    "[::1]:8080": 1 // IPv6 地址需要用方括号括起来
				  }
				  ```
				- 键是节点的地址（`IP/域名:端口`），值是该节点的权重。
		- `pass_host`：请求转发到上游时的 `Host` 字段。
		  collapsed:: true
			- `pass`：APISIX 会原样保留客户端请求中的 `Host` 头，并直接转发给后端。
			  collapsed:: true
				- **例子：**用户访问 `http://www.my-api.com/users`，请求头里的 `Host` 是 `www.my-api.com`。APISIX 转发给后端 `192.168.1.100` 时，`Host` 仍然是 `www.my-api.com`。
			- `node`：APISIX 会将 `Host` 头改写为所选后端节点的地址。
			  collapsed:: true
				- **例子：**用户访问 `http://www.my-api.com/users`，APISIX 将请求转发到 `httpbin.org:443`。此时，发往后端的请求头 `Host` 会被修改为 `httpbin.org`。
		- `scheme`：这个参数定义了 APISIX 与后端服务通信（转发请求）时使用的协议。
		  collapsed:: true
			- `http` / `https`：最常见的 Web 服务。
			- `grpc` / `grpcs`：用于 gRPC 协议的微服务。
			- `tcp` / `udp` / `tls`：代理数据库（如 MySQL）、消息队列等非 HTTP 协议的服务。
			- 默认值为 `http`。
		- `name`：一个可选的、人类可读的描述性字符串，为 Upstream 提供一个方便记忆和识别的名称。
		  collapsed:: true
			- `name` 和 `id` 的区别：
			  collapsed:: true
				- `id` 是系统用于唯一识别和引用资源的主键（机器标识），例如 Route 或 Service 会通过 `upstream_id` 来引用它，因此必须全局唯一且保持稳定。
				- `name` 仅作为资源的标签，用于描述和标识，既不强制唯一，也不会被其他配置引用，可随时修改而不影响关联配置。
		- `timeout`：设置与上游通信的超时时间（单位：秒），用于防止请求因后端响应过慢而无限期阻塞。
		  id:: 68db71a3-1c3e-4e02-b57d-956afea2dfa9
		  collapsed:: true
			- 包含三个子项：
				- `connect`：连接上游的超时时间。
				- `send`：向上游发送请求的超时时间。
				- `read`：从上游读取响应的超时时间。
-