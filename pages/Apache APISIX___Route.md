- **Route 是什么？**
	- 路由定义了一系列匹配规则，当一个请求满足这些规则时，APISIX 会对这个请求执行一系列插件定义的操作，并最终把它转发给预先设定的上游服务。
- **Route API**
	- **Route 资源请求路径：**`/apisix/admin/routes/{id}?ttl=0`
	- **ID 命名规则：**APISIX 资源的 ID 长度必须在 1 到 64 个字符之间，仅包含大小写字母、数字、中划线（`-`）、下划线（`_`）和点号（`.`）。
	- **请求方法：**
		- `GET`，`/apisix/admin/routes`，无请求体参数，获取所有路由
		- `GET`，`/apisix/admin/routes/{id}`，无请求体参数，根据指定的 id 获取路由
		- `PUT`，`/apisix/admin/routes/{id}`，若干个请求体参数，根据指定的 id 创建路由
		- `POST`，`/apisix/admin/routes`，若干个请求体参数，创建路由并分配一个随机的 id
		- `DELETE`，`/apisix/admin/routes/{id}`，无请求体参数，根据指定的 id 删除路由
		- `PATCH`，`/apisix/admin/routes/{id}`，若干个请求体参数，更新指定路由的配置
		  collapsed:: true
			- 把参数的值设置为 `null` 是删除。
		- `PATCH`，`/apisix/admin/routes/{id}/{path}`，若干个请求体参数，更新指定路径下的属性
		  collapsed:: true
			- 其它的属性保持不变。
	- **请求体参数：**
		- `id`：路由的唯一标识符。方便以后管理这条路由。
		- `uri`：路由的匹配规则。当客户端请求的 URL 路径符合你定义的这个规则时，该路由配置就会被命中并生效。
		- `upstream`：上游服务配置。一旦请求成功匹配到该路由，API 网关就会根据这个 upstream 配置，将请求转发给真正提供业务逻辑的后端服务。
		- `upstream_id`：关联的上游服务 ID。表示请求会被转发到该 ID 对应的上游服务。使用前需提前创建好相应的上游。
		- `name`：路由的名称，用于标识该路由。
		- `methods`：允许匹配的 HTTP 方法（例如 `["GET", "POST"]`）。该字段定义了哪些 HTTP 动词可以触发此路由。如果不指定此数组，则默认匹配所有 HTTP 方法。
		- `status`：路由的启用状态。1 表示启用，0 表示禁用。
- **Route 怎么用？**
	- **创建一个最简单的路由：**
		- 任何访问 APISIX 的 `/ip` 路径的请求，都应该被转发到上游服务 `httpbin.org` 的 `80` 端口。
		- ```http
		  ### 创建一个最简单的路由
		  
		  PUT http://10.30.60.116:9180/apisix/admin/routes
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "id": "getting-started-ip",
		    "uri": "/ip",
		    "upstream": {
		      "type": "roundrobin",
		      "nodes": {
		        "httpbin.org:80": 1
		      }
		    }
		  }
		  ```
- **Route 的工作流程：**
	- **客户端发起请求：**客户端向 APISIX 的代理端口发送 HTTP/HTTPS 请求。
	  logseq.order-list-type:: number
	- **APISIX 接收与匹配：**APISIX 接收到请求后，根据请求的各项属性，在内部的路由库中进行匹配查找。
	  logseq.order-list-type:: number
	- **确定上游服务：**APISIX 成功匹配到对应的路由后，读取该路由配置中的上游（Upstream）信息，以确定请求需要转发到哪个后端服务实例。
	  logseq.order-list-type:: number
	- **转发请求：**APISIX 作为代理，向确定的后端服务发起新的请求。
	  logseq.order-list-type:: number
	- **接收并返回响应：**后端服务处理完请求后，将响应返回给 APISIX，APISIX 再原样返回给最初发起请求的客户端。
	  logseq.order-list-type:: number
-