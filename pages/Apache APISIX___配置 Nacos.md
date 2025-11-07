## 架构说明
	- APISIX 网关本身不存储后端微服务的具体 IP 地址，而是动态地从 Nacos "拉取" 最新的服务实例列表。
	- ### 后端微服务
		- **关键动作：**每个微服务实例在**启动时**会主动向 Nacos Server **注册 (Register)** 自身，告知服务名（如 `user-service`）、IP 地址和端口（如 `10.0.1.10:8080`）。
		- 随后，实例会定期向 Nacos 发送**心跳 (Heartbeat)**，表明自己仍在运行。若 Nacos 连续多次未收到心跳，则会将该实例从服务列表中移除。
	- ### Nacos Server（服务注册中心）
		- **核心职责：**维护一个**实时更新的服务实例列表**。它记录每个服务（如 `user-service`）当前有哪些健康实例在运行及其 IP、端口信息，并将这些数据提供给 APISIX，告诉它“**请求应转发到哪些实例**”。
	- ### APISIX 网关 (数据平面)
		- **核心职责：**作为流量入口，负责**接收请求、查询 Nacos 并转发流量**。
		- **关键动作：**
			- **接收请求：** 客户端访问 `api.yourcompany.com/user/info`。
			- **匹配路由：** APISIX 从 `etcd` 读取路由配置，发现 `/user/*` 路径应转发到名为 `user-service` 的上游 (Upstream)。
			- **服务发现：** 由于该上游配置了 `discovery_type: nacos`，APISIX 会**向 Nacos 查询** `user-service` 的所有健康实例。
			- **获取列表：** Nacos 返回当前可用实例列表，例如 `[10.0.1.10:8080, 10.0.1.11:8080]`。APISIX 会将结果在内存中短暂缓存（如 5 秒）。
			- **转发请求：** APISIX 从列表中选择一个健康实例（如 `10.0.1.10:8080`），并将原始请求转发给它。
	- 这种架构的最大好处是**弹性伸缩**：
		- 当“用户服务”需要扩容时，只需启动新的实例（如 `10.0.1.12:8080`）。该实例会自动注册到 Nacos，APISIX 在下次查询时即可自动发现并开始转发流量，全程无需修改配置或重启网关。
- ## 配置
	- ### Nacos 配置
		- 在 `conf/config.yaml` 中添加以下配置：
			- ```yaml
			  discovery:
			    nacos:
			      host:
			        - "http://${username}:${password}@${host1}:${port1}"
			      prefix: "/nacos/v1/"
			      fetch_interval: 30    # 默认 30 秒
			      # `weight` 是默认权重，将分配给每个没有在 Nacos 返回结果中明确指定权重的节点
			      weight: 100           # 默认 100
			      timeout:
			        connect: 2000       # 默认 2000 ms
			        send: 2000          # 默认 2000 ms
			        read: 5000          # 默认 5000 ms
			  
			  ```
		- **用默认值简化配置：**
			- ```yaml
			  discovery:
			    nacos:
			      host:
			        - "http://192.168.33.1:8848"
			  
			  ```
	- ### 上游（Upstream）配置
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/routes/1
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  Content-Type: application/json
		  
		  {
		    "uri": "/user/*",
		    "upstream": {
		      "service_name": "user-service",
		      "type": "roundrobin",
		      "discovery_type": "nacos"
		    }
		  }
		  
		  ```
		- [[Apache APISIX/Upstream]]
		- `discovery_args`：
			- `namespace_id`：指定对应服务的命名空间。默认值：`public`
			  id:: 690dcfe1-a461-4a62-9956-ebe888149abd
			- `group_name`：指定对应服务的分组。默认值：`DEFAULT_GROUP`
-