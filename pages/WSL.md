WSL 是什么？
heading:: true
	- WSL（Windows Subsystem for Linux，Windows 子系统 Linux）是微软推出的一项功能，允许你在 Windows 上以轻量方式运行原生的 Linux 环境，无需安装虚拟机或配置双系统。
	- WSL 2 使用的是由微软官方维护的真实 Linux 内核。该内核基于开源版本（如 Linux 5.15 LTS），经过裁剪和适配（包括移除冗余驱动、添加与 Windows 通信的补丁）后，由微软自行编译并发布，专为在 Windows 上的 WSL 2 环境运行而设计。
- [[Windows 与 WSL 文件互传]]
- ## 安装 WSL
	- **访问：**https://learn.microsoft.com/en-us/windows/wsl/install#offline-install
	- **访问：**https://github.com/microsoft/wsl/releases
	- 下载 .msi 文件，然后点击执行。
- ## WSL 的常用命令
	- `wsl`：启动默认的 Linux 发行版。
	- `wsl --install <Distribution Name>`：安装指定的 Linux 发行版。
	- `wsl --list --online`：列出可供安装的 Linux 发行版。
	- `wsl --list --verbose`：查看所有已安装的 Linux 发行版、它们的状态以及运行的 WSL 版本（WSL 1 或 WSL 2）。
	- `wsl --unregister <Distribution Name>`：注销并永久删除指定的发行版的所有数据、设置和软件。
- ## WSL 设置网络代理
	- Windows 上的 Clash 开启 “局域网连接” 功能。
	  logseq.order-list-type:: number
	- 打开 WSL 设置，修改网络模式为 Mirrored。
	  logseq.order-list-type:: number
	- `vim ~/.bashrc`
	  logseq.order-list-type:: number
		- ```bash
		  # --- WSL Proxy Setup for Clash (Mirrored Mode) ---
		  # 代理主机地址：在镜像模式下，直接使用 127.0.0.1 访问 Windows
		  export PROXY_HOST="127.0.0.1"
		  
		  # Clash 混合端口 (来自您的截图)
		  export PROXY_PORT="7897"
		  
		  # 1. 配置 HTTP/HTTPS 代理 (使用 Clash 的混合端口 7897)
		  export http_proxy="http://${PROXY_HOST}:${PROXY_PORT}"
		  export https_proxy="http://${PROXY_HOST}:${PROXY_PORT}"
		  
		  # 2. 配置 SOCKS5 代理（Clash 混合端口也支持 SOCKS5）
		  export all_proxy="socks5://${PROXY_HOST}:${PROXY_PORT}"
		  
		  # 3. 配置大写变量 (兼容性考虑)
		  export HTTP_PROXY=$http_proxy
		  export HTTPS_PROXY=$https_proxy
		  export ALL_PROXY=$all_proxy
		  
		  # 4. 设置 no_proxy 排除本地和内部地址
		  export NO_PROXY="localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
		  export no_proxy=$NO_PROXY
		  # ------------------------------------------------
		  ```
	- `source ~/.bashrc`
	  logseq.order-list-type:: number
	- `curl -I https://www.google.com`
	  logseq.order-list-type:: number
		- 看到这样的响应就说明成功了：
			- ```bash
			  liyuze@liyuze:~$ curl -I https://www.google.com
			  HTTP/1.1 200 Connection established
			  
			  HTTP/2 200
			  content-type: text/html; charset=ISO-8859-1
			  content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-lhYKZ7dVaWh9j_Q7Gi2kpg' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
			  accept-ch: Sec-CH-Prefers-Color-Scheme
			  p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
			  date: Fri, 24 Oct 2025 14:38:03 GMT
			  server: gws
			  x-xss-protection: 0
			  x-frame-options: SAMEORIGIN
			  expires: Fri, 24 Oct 2025 14:38:03 GMT
			  cache-control: private
			  set-cookie: AEC=AaJma5uUM_mmvNptSgl4IOAmtGcaegsfDQouFefB0xtFa7UHc0yg9BT6EA8; expires=Wed, 22-Apr-2026 14:38:03 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
			  set-cookie: NID=526=VeTL1ZN8yweuJp2OnR2Q_iMB6atDBvFoHZkGaKSu2iJSApm-_lolj5RgtxDpNFXhZ_mXrLw2Wwlv58Tv0X2xgVGOqWoTGsohJuTCv5mDG-VkhEB2r9pUNyP7TKmiF0vuZ9OV5q4kTueH5ZlT61gPGX2myL209t1qOhAIRHmOYa681H29TNLWz1KTWa7M7C9o7wWsKeL3cC2M3aVArMLg; expires=Sat, 25-Apr-2026 14:38:03 GMT; path=/; domain=.google.com; HttpOnly
			  alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
			  ```
-