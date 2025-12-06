## 作用
	- 当 APISIX 位于反向代理之后时，代理是发起请求的客户端。`real-ip` 插件允许 APISIX 通过 HTTP 头或 HTTP 查询字符串中传递的 IP 地址来设置客户端的真实 IP。
- ## 属性
	- `source`：插件要从哪个变量中取出客户端的地址？
	  collapsed:: true
		- 一个内置的 APISIX 变量或 NGINX 变量，例如 `http_x_forwarded_for` 或 `arg_realip`。
		- 该变量的值应为一个有效的 IP 地址，代表客户端的真实 IP 地址，可附带一个可选的端口。
	- `trusted_addresses`
	  collapsed:: true
		- 只有来自 `trusted_addresses` 配置中的地址（支持 IP 和 CIDR）发送的 `X-Forwarded-*` 头才会被信任，并传递给插件或上游。
		- 如果未配置 `trusted_addresses`，或者 IP 不在配置的地址范围内，所有 `X-Forwarded-*` 头都将被可信值覆盖。
	- `recursive`：开启后，它会沿 `X-Forwarded-For` 链条向前查找“最后一个不在 `trusted_addresses` 里的 IP”，把它当成真实客户端，再写回 `$remote_addr`。
	  collapsed:: true
		- 可以把 `X-Forwarded-For` 想象成一串“经过的站点名单”，前面都是代理服务器，最后一个不是代理的才是真实乘客（客户端）。
		- 把 `recursive` 开启后，APISIX 会顺着这串名单从后往前找，跳过所有你信任的“中转站”（代理），最终把那个真正的“第一手来的人”的 IP 捞出来作为真正的客户端地址。
-