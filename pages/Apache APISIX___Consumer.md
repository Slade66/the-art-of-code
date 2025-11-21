- **Consumer 是什么？有什么用？**
	- 消费者是服务的使用者，是“API 调用者”的身份，它代表“谁在调用”。
	- 在 APISIX 里，Consumer 表示一个具体的“客户端/用户/应用”，比如：
		- 某个移动 App
		- 某个后端服务
		- 某个合作方客户
	- `Consumer` 的核心目的是“识别调用者的身份 + 绑定认证 / 限流等插件配置”，用于确定谁在使用你的 API，并对不同身份进行差异化管理。
	- 所有认证类插件（如 `key-auth`、`basic-auth`、`jwt-auth`、`ldap-auth` 等）的最终目的，都是将当前请求映射到一个已知的 `Consumer` 身份。
- **Consumer API**
	- **消费者资源请求地址：**`/apisix/admin/consumers/{username}`
	- **请求方法：**
		- `GET`，`/apisix/admin/consumers`，无需请求参数，获取全部的消费者
		- `GET`，`/apisix/admin/consumers/{username}`，无需请求参数，根据用户名获取指定的消费者
		- `PUT`，`/apisix/admin/consumers`，若干个请求参数，创建一个新的消费者
		- `DELETE`，`/apisix/admin/consumers/{username}`，无需请求参数，根据用户名删除指定的消费者
	- **请求体参数：**
		- `username`：消费者的名称。
		- `plugins`：用于配置消费者的认证方式及其对应的私有凭证信息，例如指定认证方式为密钥认证（`key-auth`）并设置具体的密钥（`key`）。
		  collapsed:: true
			- **与路由上的 `plugins` 参数对比**
				- 要理解 `Consumer` 中 `plugins` 参数的作用，关键是将其与 `Route` 中的同名参数进行对比。虽然名称相同，但它们的角色截然不同。
				- 在 `Consumer` 中，`plugins` 用于存放与消费者身份绑定的认证信息，例如 API 密钥或 JWT 密钥，相当于“身份凭证”。
				- 在 `Route` 中，`plugins` 用于定义准入规则或启用功能，例如“此路由需要检查 API 密钥”。它关注的是规则本身，而不涉及密钥具体内容。
				- 这就是为什么在路由上启用认证时，插件配置通常为空对象 (`{}`)：
					- ```json
					  {
					    "uri": "/profile",
					    "plugins": {
					      "key-auth": {}
					    }
					  }
					  ```
					- 这里的 `key-auth: {}` 意思就是“打开门禁，开始检查所有来访者的工牌”。但规则本身并不关心具体的工牌信息是什么。
				- 简而言之，一个负责定义“钥匙”，另一个负责规定“门上要有锁”。
	- **示例：**
		- **创建一个消费者：**
		  id:: 68d639f4-fc38-4393-b280-ed94f9856624
			- ```http
			  PUT http://10.30.60.116:9180/apisix/admin/consumers
			  Content-Type: application/json
			  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
			  
			  {
			    "username": "liyuze",
			    "plugins": {
			      "key-auth": {
			        "key": "123456"
			      }
			    }
			  }
			  ```
-