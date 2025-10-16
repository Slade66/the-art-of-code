- **有什么用？**
	- Apache APISIX 的 `proxy-rewrite` 插件在请求到达上游服务（Upstream）前，修改请求的内容，实现了客户端与后端服务的解耦，从而满足后端服务的特定期望。
	- **可修改的关键内容：**
		- **修改 HTTP 方法：**
			- **举例：**主要用于解决兼容性问题。例如，客户端使用 `PUT` 方法更新资源，但某个旧版后端服务只支持 `POST` 方法。此时可在网关层将 `PUT` 请求动态修改为 `POST`，从而在不改动任何一方代码的情况下解决问题。
		- **重写请求目标：**
			- **路径重写：**
				- **举例：**将客户端访问的 `api/users/123` 转换为后端服务能理解的 `/v1/internal/users?id=123`，以此隐藏内部实现细节，统一对外 API 风格。
			- **域名重写：**
				- **举例：**客户端请求的公共域名是 `api.public.com`，网关需要将其转发到内部名为 `user-service.internal.local` 的服务上。此时，就需要将 `Host` 头也修改为内部域名，确保请求被正确接收。
		- **修改请求头：**
			- **举例：**最常见的用途是与认证插件（如 `jwt-auth`）联动。在身份验证通过后，将从 Token 中解析出的用户信息（如 `user_id`）添加到一个新的请求头 `X-User-ID` 中再转发给后端。这样，后端服务就可以直接从请求头中获取可信的用户信息，无需重复解析 Token。
- **参数解释：**
	- `uri`
		- **解释：**用于将请求 URI 直接重写为指定的新路径。它支持 NGINX 变量，可以动态获取请求信息。在 `regex_uri` 也配置时，此参数具有更高的优先级。
		- **举例：**`"uri": "/new/path/v2"`
			- 如果客户端请求 `https://example.com/old/path`，经过此插件配置后，APISIX 转发给上游服务的 URI 将会变成 `https://upstream-service/new/path/v2`，而与原始路径无关。
	- `method`
		- **解释：**用于修改请求的 HTTP 方法。例如，将一个 `GET` 请求在转发给上游服务时改为 `POST`。
		- **举例：**`"method": "POST"`
			- 客户端可能发送了一个查询数据的 `GET` 请求到网关，但上游服务只接受 `POST` 请求进行处理。配置此项后，APISIX 会自动将请求方法从 `GET` 改为 `POST` 后再转发。
	- `regex_uri`
		- **解释：**使用正则表达式来匹配并重写 URI。它是一个键值对数组，键是用于匹配原始 URI 的正则表达式，值是新的 URI 路径。这提供了比 `uri` 更灵活的重写能力，常用于处理版本兼容性。
		- **举例：**`["^/api/v1/(.*)", "/api/v2/legacy/$1"]`
			- 如果一个请求的 URI 是 `/api/v1/users/info`，它将被重写为 `/api/v2/legacy/users/info`。`(.*)` 捕获的内容会被 `$1` 引用。
	- `host`
		- **解释：**用于设置请求的 `Host` 请求头。当您需要将请求代理到与原始域名不同的内部服务主机时非常有用。
		- **举例：**`"host": "internal.service.com"`
			- 所有经过此插件的请求，其 `Host` 请求头都会被设置为 `internal.service.com`。
	- `headers`
		- **解释：**用于修改请求头的复杂对象，可以同时执行添加、设置和移除操作。它的子属性会按 `add` -> `remove` -> `set` 的顺序执行。
		- `headers.add`
			- **解释：**用于向请求中追加新的请求头。如果请求头已存在，新值将被附加在原有值之后。
			- **举例：**
				- ```json
				  "headers": {
				    "add": {
				      "X-Request-ID": "$id"
				    }
				  }
				  ```
				- 为每个请求添加一个名为 `X-Request-ID` 的请求头，其值由 NGINX 变量 `$id` 动态生成。
		- `headers.set`
			- **解释：**用于设置请求头，会覆盖任何已存在的值。
			- **举例：**
				- ```json
				  "headers": {
				    "set": {
				      "Authorization": "Bearer token_from_plugin"
				    }
				  }
				  ```
				- 强制将 `Authorization` 请求头的值设置为指定内容，覆盖客户端带来的原始值。
		- `headers.remove`
			- **解释：**用于从请求中移除指定的请求头。
			- **举例：**
				- ```json
				  "headers": {
				    "remove": ["Cookie", "Accept-Encoding"]
				  }
				  ```
				- 在请求转发给上游服务之前，移除 `Cookie` 和 `Accept-Encoding` 请求头。
-