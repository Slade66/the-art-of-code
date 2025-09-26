- **为什么要限流？**
  id:: 68d646dc-ccdd-43d9-8dc2-14a4550d6460
	- **防止滥用：**部分用户或程序可能在短时间内发送大量请求，限流能确保资源被合理、均衡地使用。
	- **抵御爬虫：**过度的网络爬虫会给服务器带来额外压力，影响正常服务。
	- **防御 DDoS 攻击：**通过限制请求速率，避免攻击者的海量请求耗尽服务资源，从而保障正常用户的访问。
	- **控制成本：**在按使用量计费的云服务环境中，限流能防止因突发流量导致费用飙升。
	- **保持服务稳定：**突如其来的流量洪峰可能压垮后端，限流可以有效保护系统的稳定性。
- **limit-count 有什么用？**
	- `limit-count` 插件用于基于固定时间窗口的速率限制。它会统计指定时间段内的请求次数，超出预设配额的请求将被拒绝。
	- 为了方便客户端了解当前的限制状态，该插件会在响应头中附加以下信息：
		- `X-RateLimit-Limit`：当前窗口内的总请求配额。
		- `X-RateLimit-Remaining`：当前窗口内剩余的可用请求次数。
		- `X-RateLimit-Reset`：距离当前窗口结束、配额重置还需的秒数。
- **limit-count 怎么用？**
	- **在路由上启用 limit-count 插件：**
		- ```http
		  PATCH http://10.30.60.116:9180/apisix/admin/routes/test-route
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "plugins": {
		      "limit-count": {
		        "count": 2,
		        "time_window": 10,
		        "rejected_code": 503
		      }
		    }
		  }
		  ```
	- **在路由上禁用 limit-count 插件：**
		- ```http
		  PATCH http://10.30.60.116:9180/apisix/admin/routes/test-route
		  Content-Type: application/json
		  X-API-KEY: edd1c9f034335f45453292ad8625c8f1
		  
		  {
		    "plugins": {
		      "limit-count": {
		        "_meta": {
		          "disable": true
		        }
		      }
		    }
		  }
		  ```
- **limit-count 的参数：**
	- `count`：在时间窗口内允许通过的请求总数。
	- `time_window`：时间窗口的计数周期，单位为秒。
	- `key`：统计请求次数的维度。
		- `remote_addr`：按客户端 IP 限流（默认），防止单个 IP 过度请求。
		- `consumer_name`：按用户限流，与认证插件配合，为不同用户提供独立配额。
	- `rejected_code`：超出限流时返回的 HTTP 状态码。
	- `policy`：计数器存储位置。
		- `local`：存放在当前 APISIX 实例的内存中（默认）。
		- `redis` / `redis-cluster`：存放在可供所有 APISIX 实例访问的 Redis 数据库中。
-