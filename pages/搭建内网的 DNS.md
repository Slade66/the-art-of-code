## dig 和 nslookup
	- 把 DNS 想成“问路台”。`dig` 和 `nslookup` 就是两种常用的“问路器”：它们向指定的 DNS 服务器询问某个域名应该指向哪个地址，并把服务器的回答显示出来。`dig` 更现代、输出更灵活；`nslookup` 老牌、交互式使用方便。
	- ### dig
		- **语法：**`dig @服务器 域名 类型 +选项`
		- 查询本机 DNS 服务器的 A 记录并仅输出 IP：`dig @127.0.0.1 nantian.com A +short`
	- ### nslookup
		- **语法：**`nslookup 域名 DNS服务器`
		- **例子：**`nslookup nantian.com 127.0.0.1`
- ## NRPT
	- Name Resolution Policy Table 规则，也就是 Windows 的 按域名定向 DNS 功能。这是 Windows 内置的一种 Split DNS（按域名路由 DNS） 机制。
	- 你没有修改系统的全局 DNS，你只是让某个域名强制走某个指定的 DNS 服务器。
	- **Add-DnsClientNrptRule**
		- 匹配某个域名及其所有子域名（即前缀匹配）：
			- ```powershell
			  # 注意：Namespace 参数里加了个点
			  Add-DnsClientNrptRule -Namespace ".nt.cn" -NameServers "10.30.60.116"
			  ```
		- 当访问 nt.cn 这个域名及其子域名时，使用 10.30.60.116 这个 DNS 进行解析。其他域名依然走默认系统 DNS，不受影响。
	- **直接运行 nslookup 不走 NRPT**
		- 不指定 DNS 服务器 IP，它直接询问系统的 DNS Server，而不是 NRPT 规则中定义的 DNS。
		- `nslookup` 是一个调试工具，它倾向于直接读取网卡上配置的 DNS 服务器，而忽略组策略或 NRPT 表。
		- **如何用 nslookup 验证 NRPT 生效？**
			- `nslookup nt-cxfz.com 10.30.60.116`，手动指定 DNS 服务器。
	- **查看 NRPT：**`Get-DnsClientNrptRule`
	- **把域名解析成 IP：**`Resolve-DnsName nt-cxfz.com`
		- Resolve-DnsName 会走 NRPT。
	- **刷新 DNS 缓存：**`Clear-DnsClientCache`
	- **ping 的解析顺序：**
		- hosts 文件
		  logseq.order-list-type:: number
		- DNS 缓存
		  logseq.order-list-type:: number
		- NRPT 命名空间规则
		  logseq.order-list-type:: number
		- 默认系统 DNS
		  logseq.order-list-type:: number
- CoreDNS 的配置文件 `Corefile`：
	- ```
	  .:53 {
	      hosts {
	          10.30.60.116 git.nantian.com
	          
	          fallthrough
	      }
	      forward . 114.114.114.114 8.8.8.8
	      log
	      errors
	  }
	  ```
- ## 在 Linux 上设置分域解析
	- split DNS 不是 `resolv.conf` 能干的事情，只有 `systemd-resolved` 可以按域名分流。
	- **流程：**
		- 程序查询域名 → 查询 `/etc/resolv.conf`
		  logseq.order-list-type:: number
		- `/etc/resolv.conf` 指向 127.0.0.53
		  logseq.order-list-type:: number
		- 请求打到 `systemd-resolved`
		  logseq.order-list-type:: number
		- `systemd-resolved` 检查你的分域规则
		  logseq.order-list-type:: number
		- 决定走哪个 DNS
		  logseq.order-list-type:: number
			- 你访问 nt.cn → `systemd-resolved` 把请求分流给你的内网 DNS（10.x.x.x）
			  logseq.order-list-type:: number
			- 你访问 google.com → `systemd-resolved` 把请求丢给你的默认公网 DNS（8.8.8.8）
			  logseq.order-list-type:: number
	- **配置：**
		- ```bash
		  systemctl status systemd-resolved
		  systemctl start systemd-resolved
		  mkdir -p /etc/systemd/resolved.conf.d
		  vim /etc/systemd/resolved.conf.d/ntcn.conf
		  systemctl restart systemd-resolved
		  ```
	- **`ntcn.conf` 的内容：**
		- ```ini
		  [Resolve]
		  DNS=10.30.60.116
		  Domains=~nt.cn
		  ```
		- `DNS=` 指定访问 `Domains` 定义的域名时，使用这个 DNS 服务器
		- `~nt.cn` 表示“只匹配 nt.cn 这个域名及它的子域”
		- 就像一句规则：“只要域名是 nt.cn，就去 10.0.0.2 这个内网 DNS 问一下。”
	- **⚠️注意：`/etc/resolv.conf` 必须由 `systemd-resolved` 接管**
		- **执行：**`ls -l /etc/resolv.conf`
		- **应该是：**`/etc/resolv.conf -> /run/systemd/resolve/stub-resolv.conf`
		- **否则执行：**
			- ```bash
			  ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
			  systemctl restart systemd-resolved
			  ```
		- **为什么 `systemd-resolved` 需要占用 `/etc/resolv.conf`？**
			- Linux 的所有程序（curl、ping、docker、git、jenkins…）都只能从 `/etc/resolv.conf` 知道“我应该找谁解析域名”。
			- `systemd-resolved` 会开一个 本地 DNS 服务器：`127.0.0.53:53`，系统里所有程序都应该先来找它，它再帮你判断到底要走哪个真实 DNS，于是它希望 `resolv.conf` 指向它的 DNS。
			- **为什么不是 127.0.0.1？为什么是 127.0.0.53？**
				- 因为 127.0.0.1 有可能已经被别的服务占用了。
				- 而 127.x.x.x 本来就是 loopback 范围，使用 127.0.0.53 不会被抢占，不会干扰用户自己运行的 DNS 服务，所以它选了一个看起来奇怪但“安全、不冲突”的地址。
		- **为什么要 `ln -sf`？**
			- 因为很多工具会破坏 `/etc/resolv.conf`。
			- 例如：Docker 会把 `/etc/resolv.conf` 替换成：
				- ```ini
				  nameserver 127.0.0.11   # Docker 内置 DNS
				  ```
			- 这样所有 DNS 流量都绕过 `systemd-resolved`，你的 split DNS 配置当然就不会生效。
			- **`ln` 的作用：**给一个文件创建“别名”（硬链接）或“快捷方式”（软链接），它并不会复制文件，而是创建一个“引用指针”。
				- `-s`：表示创建的是软链接。文件本身不存在，指向另一个真实文件，如果目标文件变了，软链接指向不变，删除软链接不会影响原文件。
				- `-f`：强制覆盖已有文件。没有这个参数时，如果 `/etc/resolv.conf` 已经存在，会报错，因为系统不允许覆盖。加上 `-f` 之后，如果目标文件存在，就删除它再创建软链接。
			- 意思是：把 `/etc/resolv.conf` 变成一个快捷方式，指向真实 DNS 配置文件 `stub-resolv.conf`。
			-
			-
-