- **目标：实现 `go get` 命令对内网私有代码库的拉取。**
- ## VCS 选型
  collapsed:: true
	- 技术选型的首要原则是“适度设计”。
	- GitLab CE 很耗内存，要 4G以上。而 Gitea 与 Gogs 均采用 Go 语言编写，极低资源占用（100MB 左右），Gitea 作为 Gogs 的社区驱动分支，近年来在活跃度、功能迭代速度已全面超越 Gogs。
	- Go get 要求基础设施必须严格遵循 Go 工具链的元数据发现协议。Gitea 的强大之处在于，它原生内置了对 go get 这一协议的支持。只要正确配置了 Gitea 的外部 URL（ROOT_URL），它就能自动为仓库页面响应正确的 `<meta>` 标签，无需用户手动搭建额外的元数据服务器。这正是我们选择 Gitea 而非简单的 Git Server 的核心原因之一。
- ## 操作步骤
  collapsed:: true
	- 安装 Docker：`curl -fsSL https://get.docker.com | bash`
	  logseq.order-list-type:: number
	- 创建配置文件：
	  logseq.order-list-type:: number
		- `mkdir gitea && cd gitea`
		  logseq.order-list-type:: number
		- `.env`
		  logseq.order-list-type:: number
			- ```
			  DB_USER=gitea
			  DB_PASS=Aa123456.
			  DB_NAME=gitea
			  ```
			- `docker compose` 启动时默认会自动查找同级目录下的 `.env` 文件，并将其中定义的变量（如 `DB_USER`, `DB_PASS`）注入到 `docker-compose.yml` 中引用了 `${变量名}` 的地方。
		- `nginx.conf`
		  logseq.order-list-type:: number
			- ```nginx
			  user  nginx;
			  worker_processes  auto;
			  
			  error_log  /var/log/nginx/error.log notice;
			  pid        /var/run/nginx.pid;
			  
			  events {
			      worker_connections  1024;
			  }
			  
			  http {
			      include       /etc/nginx/mime.types;
			      default_type  application/octet-stream;
			  
			      # 日志格式
			      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
			                        '$status $body_bytes_sent "$http_referer" '
			                        '"$http_user_agent" "$http_x_forwarded_for"';
			      access_log  /var/log/nginx/access.log  main;
			  
			      sendfile        on;
			      keepalive_timeout  65;
			      gzip  on;
			  
			      # 上游 Gitea 服务
			      upstream gitea_backend {
			          server server:3000;
			      }
			  
			      server {
			          listen 80;
			          # 填写你的服务器 IP
			          server_name 119.13.124.137; # 如果有域名，这里填域名
			  
			          # ------------------------------------------------------------
			          # 核心优化：允许大文件上传 (512M)
			          # ------------------------------------------------------------
			          client_max_body_size 512M;
			  
			          # ------------------------------------------------------------
			          # 核心优化：防止 Git Clone 大仓库超时
			          # ------------------------------------------------------------
			          proxy_connect_timeout 600s;
			          proxy_send_timeout 600s;
			          proxy_read_timeout 600s;
			  
			          location / {
			              proxy_pass http://gitea_backend;
			  
			              # 透传真实 IP 和协议
			              proxy_set_header Host $host;
			              proxy_set_header X-Real-IP $remote_addr;
			              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			              proxy_set_header X-Forwarded-Proto $scheme;
			          }
			      }
			  }
			  ```
		- `docker-compose.yml`
		  logseq.order-list-type:: number
			- ```yaml
			  version: "3.8"
			  
			  networks:
			    gitea-net:
			      driver: bridge
			  
			  services:
			    # 数据库服务
			    db:
			      image: postgres:15-alpine
			      container_name: gitea_db
			      restart: always
			      environment:
			        - POSTGRES_USER=${DB_USER}
			        - POSTGRES_PASSWORD=${DB_PASS}
			        - POSTGRES_DB=${DB_NAME}
			      networks:
			        - gitea-net
			      volumes:
			        - ./postgres_data:/var/lib/postgresql/data
			      healthcheck:
			        test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
			        interval: 10s
			        timeout: 5s
			        retries: 5
			  
			    # Gitea 应用服务
			    server:
			      image: gitea/gitea:latest
			      container_name: gitea_app
			      restart: always
			      environment:
			        - USER_UID=1000
			        - USER_GID=1000
			        - GITEA__database__DB_TYPE=postgres
			        - GITEA__database__HOST=db:5432
			        - GITEA__database__NAME=${DB_NAME}
			        - GITEA__database__USER=${DB_USER}
			        - GITEA__database__PASSWD=${DB_PASS}
			      networks:
			        - gitea-net
			      ports:
			        # SSH 端口映射：宿主机 2222 -> 容器 22
			        # 必须避开服务器自带的 SSH (22端口)
			        - "2222:22"
			      volumes:
			        - ./gitea_data:/data
			        - /etc/timezone:/etc/timezone:ro
			        - /etc/localtime:/etc/localtime:ro
			      depends_on:
			        db:
			          condition: service_healthy
			  
			    # Nginx 接入层
			    nginx:
			      image: nginx:alpine
			      container_name: gitea_nginx
			      restart: always
			      ports:
			        - "80:80"
			      networks:
			        - gitea-net
			      volumes:
			        # 挂载我们刚才创建的配置文件
			        - ./nginx.conf:/etc/nginx/nginx.conf:ro
			      depends_on:
			        - server
			  ```
	- 启动服务：`docker compose up -d`
	  logseq.order-list-type:: number
	- Gitea 配置：
	  logseq.order-list-type:: number
		- 打开浏览器，访问：`http://Nginx 的地址`
		  logseq.order-list-type:: number
		- 数据库设置：系统已自动读取环境变量，不需要动，保持默认即可。
		  logseq.order-list-type:: number
		- 一般设置：
		  logseq.order-list-type:: number
			- 服务器域名：填服务器的 IP 或域名。
			  logseq.order-list-type:: number
			- SSH 服务端口：`2222`（Docker 映射的是 2222）。
			  logseq.order-list-type:: number
			- HTTP 服务端口：`3000`（实际访问的是 Nginx 的 80 端口，它会负责把 80 转过来这）
			  logseq.order-list-type:: number
			- Root URL：`http://服务器的 IP 或域名/`
			  logseq.order-list-type:: number
			- **注意：**如果配置了域名，Nginx 的 `server_name` 和 Gitea 的 `ROOT_URL` 最好都统一改成域名。
			  logseq.order-list-type:: number
		- 管理员设置：立即创建一个管理员账号，否则第一个注册的人就是管理员。
		  logseq.order-list-type:: number
		- 点击 “立即安装”。
		  logseq.order-list-type:: number
	- 客户端配置：
	  logseq.order-list-type:: number
	  这是决定 `go get` 能否成功的关键，需要在每一台需要拉取代码的开发机上执行。
		- 配置 Go 环境变量：
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  # 1. 设置 GOPRIVATE
			  # 告诉 Go：凡是 119.13.124.137 开头的包，不要走 goproxy.cn，直接去源站拉取
			  go env -w GOPRIVATE="119.13.124.137"
			  
			  # 2. 设置 GOINSECURE
			  # 告诉 Go：这个 IP 没有合法的 HTTPS 证书（因为你是 IP 访问且可能是 HTTP），允许通过不安全的 HTTP 获取元数据
			  go env -w GOINSECURE="119.13.124.137"
			  ```
		- 配置 SSH 密钥（解决拉代码的鉴权问题）：
		  logseq.order-list-type:: number
			- 把 `C:\Users\liyuz\.ssh` 目录下的公钥（`id_rsa.pub` 或 `id_ed25519.pub`）上传到 Gitea。
			  logseq.order-list-type:: number
			- 验证 SSH 连通性：`ssh -p 2222 git@服务器地址`
			  logseq.order-list-type:: number
				- 如果看到 `Hi there, You've successfully authenticated...`，说明 SSH 通了。失败则说明防火墙拦截了 2222 端口或 Key 没配对。
				  logseq.order-list-type:: number
		- 配置 Git URL 自动替换：
		  logseq.order-list-type:: number
			- 当你执行 `go get 119.13.124.137/user/repo` 时，Go 会先用 HTTP 协议去询问服务器。但是下载代码时，我们希望它强制走 SSH，这样就不用输入密码，这样就不会因为 HTTP 鉴权失败报错。
			  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  # 全局告诉 Git：
			  # 当你要访问 http://119.13.124.137/ 时，
			  # 自动替换成 ssh://git@119.13.124.137:2222/
			  
			  git config --global url."ssh://git@119.13.124.137:2222/".insteadOf "http://119.13.124.137/"
			  ```
	- 发布与引用
	  logseq.order-list-type:: number
		- 发布：
		  logseq.order-list-type:: number
			- 在 Gitea 上创建仓库。
			  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  # 创建目录
			  cd A:\Users\liyuz\Desktop
			  mkdir my-utils
			  cd my-utils
			  
			  # 初始化 Go 模块 (注意：模块名必须包含 IP/域名)
			  # 格式: go mod init <IP/域名>/<用户名>/<仓库名>
			  # 假设你的 Gitea 用户名是 admin
			  go mod init 119.13.124.137/admin/my-utils
			  
			  # 创建一个简单的 go 代码文件
			  ...
			  
			  # Git 提交
			  git init
			  git add .
			  git commit -m "Initial commit"
			  git branch -M main
			  
			  # 添加远程仓库 (注意：这里使用 SSH 地址)
			  git remote add origin ssh://git@119.13.124.137:2222/admin/my-utils.git
			  git push -u origin main
			  ```
		- 引用：
		  logseq.order-list-type:: number
			- logseq.order-list-type:: number
			  ```bash
			  # 回到桌面，创建新项目
			  mkdir my-app
			  cd my-app
			  
			  # 初始化新项目
			  go mod init my-app
			  
			  # 拉取刚才发布的私有模块
			  go get 119.13.124.137/admin/my-utils
			  ```
			- **如果成功：**
			  logseq.order-list-type:: number
				- Go 会先发 HTTP 请求给 Gitea 询问元数据。
				  logseq.order-list-type:: number
				- Gitea 返回 meta tag，指向 `.git` 结尾的地址。
				  logseq.order-list-type:: number
				- Go 调用本地 Git 命令去 clone。
				  logseq.order-list-type:: number
				- 本地 Git 触发 `insteadOf` 规则，将 HTTP 链接转为 SSH（带端口 2222）。
				  logseq.order-list-type:: number
				- SSH 鉴权通过，代码下载完成。
				  logseq.order-list-type:: number
				- `go.mod` 文件中会出现 `require 119.13.124.137/admin/my-utils v0.0.0-xxxx`。
				  logseq.order-list-type:: number
- ## 常见错误
  collapsed:: true
	- `go: unrecognized import path ... reading http://...?go-get=1: 502 Bad Gateway`
	  collapsed:: true
		- **问题：**中间代理出错。
		- **原因 1：**
		  id:: 692a6f78-ee6b-4537-a43b-544200fbaeb0
			- 检查 Nginx 有没有访问日志，如果没有就是本地的翻墙代理软件出了问题。
			- **Clash 转发原理：**
				- 当你开启 Clash（或任何代理软件）时，命令行终端通常会被自动注入 `HTTP_PROXY` 和 `HTTPS_PROXY` 环境变量。
				- **Go get 发起请求**：Go 试图访问 `http://nt-cxfz.com/...`
				- **被代理拦截**：因为有环境变量，请求被 Clash 拦截。
				- **Clash 转发**：Clash 把请求发给了你的代理服务器（例如位于香港或美国的 VPS）。
				- **公网寻址**：代理服务器在公网上寻找 `nt-cxfz.com`。
				- **失败：**由于代理服务器在公网，无法解析内网域名（或解析出错误的公网 IP），甚至直接找不到该域名，最终导致代理服务器无法连接目标 IP，返回 502。
			- **解决方案：**
				- **将内网域名加入“不走代理”的名单：**
					- 打开 Clash -> Settings (设置) -> System Proxy (系统代理) -> 绕过域名/IP。
					- 确保列表里包含以下内容：`*.nt-cxfz.com` (显式添加你的内网域名)
				- **临时清除当前终端的系统代理：**
					- ```powershell
					  # 1. 查看系统环境变量中的代理设置
					  Get-ChildItem Env: | Where-Object { $_.Name -like "*PROXY*" }
					  
					  Name                           Value
					  ----                           -----
					  HTTP_PROXY                     http://127.0.0.1:7897
					  HTTPS_PROXY                    http://127.0.0.1:7897
					  
					  # 2. 手动清空
					  $env:HTTP_PROXY=""
					  $env:HTTPS_PROXY=""
					  $env:ALL_PROXY=""
					  ```
	- `go: unrecognized import path ... parse https://...?go-get=1): no go-import meta tags ()`
	  collapsed:: true
		- 执行 `curl.exe -k https://git.nantian.com/admin/my-logger?go-get=1`：查看返回的内容中是否包含 `<meta name="go-import" ...>`。
		  collapsed:: true
			- 在 Windows 上使用 curl 有两个坑：一个是真正的 cURL，一个是 PowerShell `Invoke-WebRequest` 的别名。
			- Windows 10/11 自带了标准的 curl 工具。为了不让 PowerShell 拦截，你需要加上 `.exe` 后缀。
			- -k 表示允许不安全连接（忽略证书错误）
		- **发生了什么？**
			- 当执行 `go get` 时，Go 工具链的默认逻辑是 **“优先尝试 HTTPS”**。
			- **期望的失败**：在内网环境中，通常没配 SSL（端口 443 没开）。Go 发起 HTTPS 请求，理应收到 `Connection Refused`（连接被拒绝）。一旦收到这个特定的网络错误，Go 就会**自动降级**尝试 HTTP（端口 80）。
			- **实际的情况**：Go 发起了 HTTPS 请求，但是它**没有收到“连接拒绝”**，而是**连接成功了**，并且收到了一个 HTTP 响应（比如 404，或者 502，或者一个空白页），但这个页面里没有 `<meta>` 标签。
			- **结局**：Go 认为：“既然 HTTPS 连通了，那我就以 HTTPS 的结果为准”。于是它停止尝试 HTTP，直接报错“找不到 meta 标签”。
			- 端口 443 被其它服务占用，返回了无关的信息，阻断了 Go 的自动降级。
		- **解决方案：**
			- 既然 Go 那么想用 HTTPS，我们就给它一个假的 HTTPS。这样配合 `GOINSECURE` 就能完美工作。
			- **生成自签名证书（如果其它服务已经有证书，可以直接复用，这一步可以跳过）：**
			  logseq.order-list-type:: number
				- ```bash
				  openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 \
				    -subj "/C=CN/ST=Fuzhou/L=Fuzhou/O=Dev/CN=nt-cxfz.com" \
				    -keyout server.key -out server.crt
				  ```
			- **修改 `nginx.conf`**（增加 443 监听和转发）：
			  logseq.order-list-type:: number
				- ```nginx
				  # [新增] Gitea 的 HTTPS 支持，为了满足 Go 工具链优先访问 HTTPS 的癖好
				  server {
				    listen 443 ssl;
				    server_name nt-cxfz.com;
				  
				    # 或者复用你现有的证书 (即使证书域名不匹配，因为客户端配了 GOINSECURE，所以没关系)
				    ssl_certificate /etc/nginx/certs/server.crt;
				    ssl_certificate_key /etc/nginx/certs/server.key;
				  
				    client_max_body_size 512M;
				  
				    location / {
				      proxy_pass http://gitea_backend;
				      proxy_set_header Host $host;
				      proxy_set_header X-Real-IP $remote_addr;
				      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				      proxy_set_header X-Forwarded-Proto $scheme;
				    }
				  }
				  ```
			- **把证书目录挂载进去，并暴露 443 端口，然后重启 Nginx。**
			  logseq.order-list-type:: number
				- 之前：HTTPS 请求 -> 默认规则 -> 其它平台 -> 无 Meta 标签 -> **报错**。
				- 现在：HTTPS 请求 -> `nt-cxfz.com` 规则 -> Gitea -> 返回正确的 Meta 标签 -> **成功**。
				- 配合你本地的 `go env -w GOINSECURE=nt-cxfz.com`，Go 不去验证 SSL 证书的合法性，哪怕这个证书是过期的、或者是颁发给 `google.com` 的，统统放行，只要能建立 HTTPS 连接就行。愉快地完成元数据解析，然后转交 Git 去 SSH 拉取代码。
	- `terminal prompts disabled`：
	  collapsed:: true
		- 说明 Git 正在尝试用 HTTP 下载并且需要输密码。
		- 检查 Git 的 `insteadOf` 命令是否执行。
	- `fatal: unable to access 'https://nt-cxfz.com/admin/commonutils.git/': schannel: failed to receive handshake, SSL/TLS connection failed`
	  collapsed:: true
		- 问题出在 **Git**（而不是 Go）对 HTTPS 证书的校验上。
		- **原因：**
			- **Git 正在尝试 HTTPS**：请注意，报错的 URL 是 **`https://`** 开头的。
			- Go 工具链发现你的服务器 支持 HTTPS，它就会**自动忽略** Gitea 返回的 `http://` 地址，强制把下载地址“升级”为 `https://`，因为它认为 HTTPS 更安全。
			- **规则漏网**：我们之前只配置了 `http://nt-cxfz.com/` 自动替换为 SSH。但是 Go 或 Git 在处理过程中（可能是因为 Nginx 里的 HTTPS 配置生效了，或者 Go 的自动升级机制），决定尝试访问 HTTPS 地址。
			- **规则未命中**：因为你只配了 `http` 的替换规则，没配 `https` 的替换规则，所以 Git 真的去连了 HTTPS。
			- **证书拒绝**：Git（使用 Windows 的 schannel 库）发现 Nginx 返回的证书是“智能告警平台”的，或者是自签名的，**它不信任，所以拒绝连接**。
		- **解决方案：**
			- ```bash
			  # 补全 HTTPS 的替换规则：之前的规则只覆盖了 http，现在加上 https
			  git config --global url."ssh://git@nt-cxfz.com:2222/".insteadOf "https://nt-cxfz.com/"
			  
			  # 全局关闭 Git 的 SSL 校验：因为你的证书是复用的/自签名的，Git 默认会拦截。我们需要关掉它。
			  git config --global http.sslVerify false
			  ```
- ## go get 到底在后台偷偷做了什么？
  collapsed:: true
	- **Go Modules 的导入路径（Import Path）**：`119.13.124.137/user/repo` 在 Go 语言里是一个逻辑标识符。
	- 简单来说，`go get` 的工作流程分为两个完全独立的阶段：**“问路”** 和 **“取货”**。
	- 阶段 1：问路（HTTP 协议）
		- 当你输入 `go get 119.13.124.137/user/repo` 时，Go 工具链并不知道这是一个 Git 仓库，还是 SVN 仓库，也不知道具体的下载地址。
		- **动作：**Go 会先发起一个普通的 HTTP GET 请求（就像浏览器访问网页一样）去访问这个网址，寻找“元数据”（库的下载地址在哪，要用哪个 VCS 去下载）。
			- 它会对模块路径发出一个 HTTP 请求：`https://git.internal.corp/dev-team/common-lib?go-get=1`。注意 URL 末尾的 `?go-get=1` 参数，这是 Go 专用的探测信号 。
		- **结果**：Gitea 服务器会返回一个 HTML 页面，里面藏着一行代码（`<meta>` 标签），告诉 Go：“你好，这个项目的代码存储在 `http://119.13.124.137/user/repo.git`，请用 `git` 工具去下载。”
			- 服务器必须返回一个包含 `<meta>` 标签的 HTML 页面。
			- 例如：`<meta name="go-import" content="git.internal.corp/dev-team/common-lib git https://git.internal.corp/dev-team/common-lib.git">`。
			- 这行代码告诉 Go：“你要找的包 `git.internal.corp/dev-team/common-lib` 实际上存储在 `https://git.internal.corp/dev-team/common-lib.git` 这个 Git 仓库中，请使用 git 工具去下载。”
	- 阶段 2：取货（Git 协议）
		- 拿到地址后，Go 就退居二线了，它会调用你电脑上的 `git` 命令来执行下载。
		- **原始动作**：Go 会对搬运工 Git 说：“嘿，去把 `http://119.13.124.137/user/repo.git` 这个仓库克隆下来。”
		- **遇到的死胡同**：
			- Git 接到命令，尝试用 HTTP 访问该地址。
			- 因为是**私有仓库**，Gitea 说：“站住，这是私有财产，请输入用户名和密码。”
			- **关键问题来了**：`go get` 是在后台自动运行的，它**没办法弹出一个窗口让你输入密码**。
			- **结局**：Git 等不到密码，直接报错，下载失败。
		- **HTTP 下载需要输密码**：由于自动化脚本没法输密码，所以 HTTP 下载私有仓库必死。
	- 阶段 3：神奇的偷天换日 ( `insteadOf`  配置)
		- 当我们配置了：`git config url."ssh://git@119...:2222/".insteadOf "http://119.../"`
		- 流程就变成了这样：
			- Go 对 Git 说：“去克隆 `http://119.13.124.137/...`”
			- Git 刚要出发，查了一下自己的配置文件，发现了一条规则：“**凡是遇到 `http://119.13.124.137/` 的请求，全部自动替换成 `ssh://git@119.13.124.137:2222/`**”。
			- **偷天换日**：Git 实际上执行的命令变成了连接 **SSH 端口**。当 Git 准备出发时，我们强制它把交通工具换成了 **SSH**（带着私钥这把钥匙进行静默认证）。
			- **验证**：SSH 连接时，Git 会自动使用你电脑里的 **SSH 私钥** 去和服务器的公钥对暗号。
			- **成功**：对暗号成功（因为你已经上传了公钥），不需要人工输入密码，代码顺利下载。
		- **SSH 下载不需要输密码**：靠密钥对验证，适合自动化。
		- **`insteadOf` 的作用**：**就是当中间人，把 Go 拿到的“死路（HTTP）”偷偷换成“活路（SSH）”。我们通过配置让它在下载代码时自动切换成了 SSH 协议免密拉取。
- ## GOPROXY
  collapsed:: true
	- 当你 `go get` 或构建项目时，Go 需要从互联网上下载模块（依赖）。GOPROXY 就是告诉 Go：去哪里下载这些模块。
	- 当开发者执行 `go get example.com/utils/logger` 时，Go 工具链并不会直接去访问 `example.com` 的版本控制系统。相反，它会遵循 `GOPROXY` 环境变量的指示，通常指向 `https://proxy.golang.org`。这是一个由 Google 维护的公共模块镜像源，它会缓存开源代码的元数据与源码压缩包。
	- GOPROXY = “Go 模块的 CDN”，作用 = “加速、缓存、提高可用性”。
	- 它解决的问题主要有：官方源太慢或网络不可达。
	- `set GOPROXY=https://goproxy.cn,direct`：
		- 意思是：先从 goproxy.cn 下载，如果找不到，就直接从源码库拉（direct）。
	- 代理服务器负责缓存：一次请求后代理会缓存模块内容，这样下次速度会极快。
	- 校验防止篡改（sumdb）：为了保证模块没被伪造或篡改，Go 使用 `GOSUMDB`（例如 sum.golang.org）进行校验。
		- 比喻：GOPROXY 给你书，GOSUMDB 提供书的“防伪码”，Go 检查防伪码，确认内容没被改过。
	- **比喻：**
		- 可以把 GOPROXY 想象成一个“图书馆”：
			- 你想借一本书（某个版本的模块）。
			- 图书馆（GOPROXY）可能已经把它缓存好了，
			- 如果没有，图书馆会帮你去出版社（GitHub / GitLab / 源码仓库）取，
			- 取到之后放进书架（缓存），别人也可以用，
			- 最后把书给你。
	- 如果没有 GOPROXY 会怎样？
		- 就像你每次借书都要跑到出版社去拿，下载慢，一些模块因为法律/网络原因无法访问。
- ## GOPRIVATE
  collapsed:: true
	- Go 默认会尝试使用 GOPROXY 获取模块，但对于私有仓库，这一步会失败。
	- Go 1.13+ 默认会去 sum.golang.org 校验哈希值，去 proxy.golang.org 拉取代码。对于内网 Gitea，这会失败。
	- 对于私有仓库而言，这种默认机制是行不通的。首先，公共代理无法访问企业内网；其次，企业也不希望内部代码被上传到公共服务器。
	- 因此，配置私有仓库的第一步是在客户端通过 `GOPRIVATE` 环境变量告知 Go 工具链：“对于某些特定的域名（如公司内网域名）下的所有包，不要走代理，不要校验公网 sumdb，直接去 Git 拉取。”。
	- 告诉 Go：属于这些域名的模块是私有的，别把它们当作公共模块，不要通过代理和校验服务器处理，直接从源码库下载，不准走代理。
	- `go env -w GOPRIVATE=git.mycompany.com,*.corp`
		- 只要模块路径属于这些域名
		- 一律跳过 GOPROXY（代理）
		- 一律跳过 GOSUMDB（校验服务器）
		- Go 直接用 git 连接内网仓库拉代码
	- 公共代理不能处理私有仓库：
		- 你公司内部有一个保险柜（私有 Git 仓库），外面的快递员（公共 GOPROXY）根本进不来保险柜所在的楼，GOPROXY 服务器访问不到内网服务器。
		- **私有仓库的代码不可能放在公共 GOPROXY 上，也不能由 GOPROXY 代为下载。**
- ## GOINSECURE
  collapsed:: true
	- go 工具链先用 HTTP 做“模块信息获取”（Go先问这个仓库在哪里？我该用 git 还是其他方式？），然后再 git clone。
	- 如果只开了一个 HTTP 接口，TLS 不安全，如果不设置 GOINSECURE，Go 会直接拒绝第一个 HTTP GET，导致根本走不到 git clone 阶段。
	- GOINSECURE 的作用是**允许不安全连接（HTTP 或无效证书）**，让“模块推断阶段”能继续跑下去，从而有机会走到 git clone。
	- `GOINSECURE` 只影响 Go 自己发出的元数据请求（HTTP GET），而不影响 git clone。
		- git clone 是外部程序（git）发起的，Go 不会修改 git 的 TLS 或证书行为。
		- git 对自签证书需要你自己配（例如 `git config http.sslVerify false`）。
	- `go env -w GOINSECURE=*.intranet.local`
		- 只要域名匹配，那么 Go 会：
			- 允许使用 HTTP
			- 忽略 HTTPS 证书无效问题
			- 不会返回 TLS 验证错误
- ## MacOS 配置
  collapsed:: true
	- 配置 Split DNS：
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  sudo mkdir -p /etc/resolver
		  sudo sh -c 'echo "nameserver 10.30.60.116" > /etc/resolver/nt.cn'
		  sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
		  ping -c 2 gitea.nt.cn
		  ping -c 2 baidu.com
		  
		  ```
	- 配置 Go 环境变量：
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  # 凡是 gitea.nt.cn 开头的包，不走 goproxy.cn 代理，直接回源
		  go env -w GOPRIVATE="gitea.nt.cn"
		  
		  # 允许 HTTPS 证书无效（自签名/域名不匹配）或降级 HTTP
		  go env -w GOINSECURE="gitea.nt.cn"
		  ```
	- 配置 SSH 访问：
	  logseq.order-list-type:: number
		- 生成公钥：`ssh-keygen -t ed25519 -C "your_mac@company.com"`，一路回车
		  logseq.order-list-type:: number
		- 复制公钥：`cat ~/.ssh/id_ed25519.pub | pbcopy`
		  logseq.order-list-type:: number
		- 上传到 Gitea：去 Gitea 网页 -> 右上角头像 -> 设置 -> SSH / GPG 密钥 -> 添加密钥。
		  logseq.order-list-type:: number
		- 测试连通性：`ssh -p 2222 git@gitea.nt.cn`
		  logseq.order-list-type:: number
	- 配置 Git 强制替换规则：
	  logseq.order-list-type:: number
		- logseq.order-list-type:: number
		  ```bash
		  # 1. 针对 HTTPS 的强制替换 (Go 默认会尝试 HTTPS)
		  # 当 Git 收到 https://gitea.nt.cn 请求时，自动替换为 ssh://git@gitea.nt.cn:2222
		  git config --global url."ssh://git@gitea.nt.cn:2222/".insteadOf "https://gitea.nt.cn/"
		  
		  # 2. 针对 HTTP 的强制替换 (防止 Go 降级尝试 HTTP)
		  git config --global url."ssh://git@gitea.nt.cn:2222/".insteadOf "http://gitea.nt.cn/"
		  
		  # 3. 关闭全局 SSL 校验 (防止 Nginx 证书不匹配导致报错)
		  git config --global http.sslVerify false
		  ```
	- 测试：`go get gitea.nt.cn/nantian/pkg`
	  logseq.order-list-type:: number
- [[搭建内网的 DNS]]
-