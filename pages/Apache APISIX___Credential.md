-
- **Credential 是什么？**
	- Consumer 表示一个 API 用户，而 Credential（凭证）则是与该用户绑定的各种认证密钥。
- **Credential 有什么用？**
	- 凭证用于保存 Consumer 的认证信息。
	- **一个 Consumer 需要多种认证方式：**
	  collapsed:: true
		- 用户 jack 同时拥有 jwt-auth 和 key-auth 两种认证方式，因此他既可以使用 JWT token，也可以使用 API Key 来访问受保护的路由。
		- ```json
		  # 创建 Consumer jack
		  {
		    "username": "jack"
		  }
		  
		  # 给 jack 添加 jwt-auth 凭证
		  {
		    "id": "cred-jack-jwt",
		    "plugins": {
		      "jwt-auth": {
		        "key": "jack-key",
		        "secret": "jack-secret"
		      }
		    }
		  }
		  
		  # 给 jack 添加 key-auth 凭证
		  {
		    "id": "cred-jack-key",
		    "plugins": {
		      "key-auth": {
		        "key": "jack-api-key"
		      }
		    }
		  }
		  ```
	- **一个 Consumer 需要某个认证插件的多个不同配置：**
	  collapsed:: true
		- 用户 alice 拥有两个 jwt-auth 凭证，以便在不同场景下使用。如果其中一个密钥泄露，只需删除对应的 credential，不会影响另一个。
		- ```json
		  # 创建 Consumer alice
		  {
		    "username": "alice"
		  }
		  
		  # 给 alice 添加第一个 jwt-auth 凭证（比如用于手机端）
		  {
		    "id": "cred-alice-jwt-1",
		    "plugins": {
		      "jwt-auth": {
		        "key": "alice-mobile",
		        "secret": "alice-mobile-secret"
		      }
		    }
		  }
		  
		  # 给 alice 添加第二个 jwt-auth 凭证（比如用于服务器端）
		  {
		    "id": "cred-alice-jwt-2",
		    "plugins": {
		      "jwt-auth": {
		        "key": "alice-server",
		        "secret": "alice-server-secret"
		      }
		    }
		  }
		  ```
	- **凭证可以独立管理，易于撤销：**
	  collapsed:: true
		- Consumer 是逻辑上的用户，通常保持不变。
		- Credential 是用户的具体认证手段，可随时新增或删除
		- 可以通过撤销某个 Credential 来禁用某个设备，而无需删除整个用户。
- **Credential API**
	- **Credential 资源请求地址：**`/apisix/admin/consumers/{username}/credentials/{credential_id}`
	- `plugins`：认证插件的配置
- **例子：**
	- 首先，创建一个未指定认证插件的 Consumer，后续再通过 Credential 配置认证插件：
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/consumers
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "username": "liyuze"
		  }
		  ```
	- 为该 Consumer 创建两个 key-auth 凭证：
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/consumers/liyuze/credentials/key-auth-one
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "plugins": {
		      "key-auth": {
		        "key": "auth-one"
		      }
		    }
		  }
		  ```
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/consumers/liyuze/credentials/key-auth-two
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "plugins": {
		      "key-auth": {
		        "key": "auth-two"
		      }
		    }
		  }
		  ```
	- 创建一个路由，并在其上启用 key-auth 插件：
		- ```http
		  PUT http://10.30.60.116:9180/apisix/admin/routes/ip
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  Content-Type: application/json
		  
		  {
		    "plugins": {
		      "key-auth": {}
		    },
		    "upstream": {
		      "nodes": {
		        "httpbin.org:80": 1
		      },
		      "type": "roundrobin"
		    },
		    "uri": "/ip"
		  }
		  ```
	- 使用 `auth-one` 和 `auth-two` 两个密钥请求该路由，都能正确返回结果：
		- ```http
		  GET http://10.30.60.116:9080/ip
		  apikey: auth-one
		  ```
		- ```http
		  GET http://10.30.60.116:9080/ip
		  apikey: auth-two
		  ```
-