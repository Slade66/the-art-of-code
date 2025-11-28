## dig 和 nslookup
	- 把 DNS 想成“问路台”。`dig` 和 `nslookup` 就是两种常用的“问路器”：它们向指定的 DNS 服务器询问某个域名应该指向哪个地址，并把服务器的回答显示出来。`dig` 更现代、输出更灵活；`nslookup` 老牌、交互式使用方便。
	- ### dig
		- `dig @服务器 域名 类型 +选项`
		- 查询本机 DNS 的 A 记录并仅输出 IP：`dig @127.0.0.1 nantian.com A +short`
	- ### nslookup
		- `nslookup 域名 服务器`
		- `nslookup nantian.com 127.0.0.1`
- ## NRPT
	- Name Resolution Policy Table 规则，也就是 Windows 的 按域名定向 DNS 功能。这是 Windows 内置的一种 Split DNS（按域名路由 DNS） 机制。
	- 你没有修改系统的全局 DNS，你只是让某个域名强制走某个指定的 DNS 服务器。
	- `Add-DnsClientNrptRule -Namespace "nt-cxfz.com" -NameServers "10.30.60.116"`
		- 当访问 nt-cxfz.com 这个域名时，使用 10.30.60.116 这个 DNS 进行解析。
		- 其他域名依然走默认系统 DNS，不受影响。
	- **nslookup 不走 NRPT**
		- 它直接询问系统的 DNS Server，而不是 NRPT 规则中定义的 DNS。
		- **如何用 nslookup 验证 NRPT 生效？**
			- `nslookup nt-cxfz.com 10.30.60.116`，手动指定 DNS 服务器。
	- **查看 NRPT：**`Get-DnsClientNrptRule`
	- **查看某域名实际解析用的 DNS：**`Resolve-DnsName nt-cxfz.com`
		- Resolve-DnsName 会走 NRPT。
	- **ping 的解析顺序：**
		- hosts 文件
		  logseq.order-list-type:: number
		- DNS 缓存
		  logseq.order-list-type:: number
		- NRPT 命名空间规则
		  logseq.order-list-type:: number
		- 默认系统 DNS
		  logseq.order-list-type:: number
	-
-