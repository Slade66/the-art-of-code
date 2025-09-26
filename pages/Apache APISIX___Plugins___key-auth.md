- **key-auth 有什么用？**
	- `key-auth`（密钥认证）通过验证客户端请求头中 `apikey` 字段的值是否为一个有效的、预先配置的 API 密钥，来决定是放行请求还是以 `401 Unauthorized` 状态拒绝请求。
- **key-auth 的使用流程：**
	- ((68d639f4-fc38-4393-b280-ed94f9856624))
	- **在路由上启用认证：**
		- 使用 `PATCH` 方法为路由添加 `key-auth` 插件，这相当于告诉该路由：“从现在起，你需要验证访客的身份！”
		- ```http
		  PATCH http://10.30.60.116:9180/apisix/admin/routes/test-route
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "plugins": {
		      "key-auth": {}
		    }
		  }
		  ```
- **key-auth 的认证流程：**
	- APISIX 匹配到路由 `/orders/123`。
	- 发现该路由启用了 `key-auth` 插件。
	- `key-auth` 插件从请求头中提取 API Key，例如 `ABCDEFG12345`。
	- 插件在“消费者档案库”中查找该 Key，发现它属于 `my-app` 这个消费者。
	- 认证成功！APISIX 确认“来访者是 `my-app`”，放行请求。如果请求中缺少 Key 或 Key 错误，APISIX 将直接拒绝请求，返回 `401 Unauthorized`。
-