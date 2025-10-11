- **jwt-auth 插件有什么用？**
	- jwt-auth 插件要求客户端在访问后端服务之前，通过 JWT 来验证自己的身份。
- **jwt-auth 插件的配置项**
	- **Consumer 或 Credential 上的配置：**
		- `key`：JWT 凭证的标识符。
		  collapsed:: true
			- 这个 `key` 的值必须被包含在 JWT 的载荷（Payload）中。
			- 一个 Consumer 可以拥有多种凭证（例如 JWT、Key-Auth），每种凭证都有各自的 `key`。
			- 当 APISIX 网关收到一个 JWT 时，会先解码并读取其中的 `key` 值，然后根据该值查找具有相同 `key` 的 `credential` 配置，从而获取对应的 `secret` 来完成签名验证。
			- 相当于账户下办理的一张“JWT 登录卡”的卡号，通过这个卡号即可找到对应的密钥。
		- `secret`：使用对称加密算法时用来签发和验证 JWT 的共享密钥。
		  collapsed:: true
			- **何时使用：**当你的 `algorithm` 设置为 `HS256` 或 `HS512` 时，该参数是必需的。如果未提供，APISIX 会自动为你生成一个。
		- `public_key`：使用非对称加密算法时用来验证 JWT 签名的公钥。
		  collapsed:: true
			- **何时使用：**当你的 `algorithm` 设置为 `RS256` 或 `ES256` 时，这个参数是必需的。
			- **工作原理：**JWT 的签发方（可能是外部的认证服务）使用私钥（Private Key）进行签名，而 APISIX 仅使用公钥（Public Key）来验证签名是否有效。这样做的好处是，签发方能够妥善保管私钥，仅将公钥提供给 APISIX，后者无需知道私钥，从而提升安全性。
		- `algorithm`：生成 JWT 签名的加密算法。默认为 `HS256`。
		  collapsed:: true
			- `HS256`、`HS512`：对称算法，使用相同的 secret 进行签名和验证，简单快捷。
			- `RS256`、`ES256`：非对称算法，使用私钥进行签名，公钥进行验证，更安全，适合更复杂的系统。
		- `base64_secret`：密钥是否为 Base64 编码。
		  collapsed:: true
			- 如果你的 `secret` 值本身是经过 Base64 编码的，请将此项设为 `true`，APISIX 在使用前会先对其进行解码。
		- `key_claim_name`：Key 在载荷中的名称。
		  collapsed:: true
			- 它定义了 APISIX 应该去 JWT Payload 中的哪个字段里寻找上面我们定义的 `key` 值。
			- 默认情况下，APISIX 会在名为 `"key"` 的字段中查找。如果认证服务生成的 Token 使用 `"sub"`（Subject）或 `"iss"`（Issuer）作为用户标识，你就可以将该参数设置为 `"sub"`，这样 APISIX 就会到 `sub` 字段中获取用户标识。
	- **Routes 或 Services 上的配置：**
		- `header`：从哪个请求头字段中提取 token。默认为 `authorization`。
	- 你可以将密钥和 RSA 密钥对安全地存放在专用的密钥管理器中，而不是以明文形式写在配置里，并通过 APISIX Secret 资源从加密的 HashiCorp Vault 中获取。
- **jwt-auth 插件的使用示例：**
	- 创建消费者：
	  logseq.order-list-type:: number
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/consumers
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "username": "liyuze"
		  }
		  ```
	- 为消费者创建 JWT 凭证：
	  logseq.order-list-type:: number
		- 告诉 APISIX，`liyuze` 这个用户将使用 JWT 进行认证，并设定他的认证信息。
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/consumers/liyuze/credentials
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  Content-Type: application/json
		  
		  {
		    "id": "cred-liyuze-jwt-auth",
		    "plugins": {
		      "jwt-auth": {
		        "key": "liyuze-key",
		        "secret": "liyuze-hs256-secret-012345678910"
		      }
		    }
		  }
		  ```
	- 创建并保护路由：
	  logseq.order-list-type:: number
		- 创建一个对外开放的路径 `/hello`，并规定所有访问这个路径的请求都必须通过 `jwt-auth` 插件的认证。
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/routes
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  Content-Type: application/json
		  
		  {
		    "id": "jwt-route",
		    "uri": "/ip",
		    "plugins": {
		      "jwt-auth": {}
		    },
		    "upstream": {
		      "type": "roundrobin",
		      "nodes": {
		        "httpbin.org:80": 1
		      }
		    }
		  }
		  ```
	- 生成一个 JWT：
	  logseq.order-list-type:: number
		- 访问 https://www.jwt.io/
		- 点击 “JWT Encoder”。
		- 选择 HS256 算法。
		- 在 Payload 里填入 `key` 与 `exp` 字段。
			- `key`：须与 APISIX 凭证中的 `key` 完全一致。
			- `exp`：须为未来的 UNIX 时间戳，用于设置有效期。
		- 将 APISIX 凭证中的 `secret` 粘贴至密钥输入框。
	- 测试 API 访问：
	  logseq.order-list-type:: number
		- ```http
		  GET http://10.30.60.116:9080/ip
		  Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJsaXl1emUta2V5IiwiZXhwIjoxNzYwODkzMjYxfQ.P8dpibA_tMapryAg-LT_aDyjxIodFVRvIttxh422h6E
		  ```
- **其它：**
	- 启用插件后，会为消费者提供一个专门用于签发数字身份证（JWT）的 API 接口。消费者需要先通过该接口获取属于自己的 JWT，这个过程会生成一个令牌（token）。之后，客户端在发起请求时必须携带该令牌，以向 APISIX 证明身份。
	- 令牌可以放在请求的 URL 查询参数、请求头或 Cookie 中。
	- 当 APISIX 收到带有 JWT 的请求时，会对令牌进行验证：它是不是我签发的？它过期了没有？它有没有被篡改过？验证通过就放行，否则直接拒绝。
	- 当消费者成功通过认证后，APISIX 会在将请求代理到上游服务之前，向请求中添加一些额外的请求头，例如 `X-Consumer-Username`（用户的用户名）、`X-Credential-Identifier`（JWT 的唯一标识）以及其它配置的自定义请求头。然后，APISIX 再将请求转发给你的后端。这样，上游服务只需从请求头中读取这些信息，就能区分不同的消费者，知道“当前是谁在访问我”，从而专注于处理业务逻辑，实现了认证与业务逻辑的解耦。
	- jwt-auth 可以与 HashiCorp Vault 配合使用，将密钥或公钥存储在 Vault 这种安全的存储中，再通过 APISIX 的 Secret 资源进行读取，从而保证安全性。
-