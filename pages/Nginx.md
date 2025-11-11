## Nginx 的架构
	- Nginx 采用了主进程和工作进程的多进程模型。
	- **主进程：**
		- 负责管理工作进程，例如启动、停止、加载配置等。
		- 它不直接处理网络请求。
	- **工作进程：**
		- 由主进程派生，数量通常等于服务器的 CPU 核心数（或你设定的值）。
		- 负责实际处理所有的客户端连接和请求。
		- Nginx 之所以高效，很大程度上是因为工作进程采用了事件驱动和非阻塞的 I/O 模型。
- ## Nginx 的配置
	- `nginx.conf` 文件是 Nginx 主配置文件，是 Nginx 启动时默认加载的配置文件，是所有配置的起点。
	- ### 全局配置
		- **作用：**它定义了 Nginx 进程本身应该如何运行、使用什么用户、在哪里记录日志等环境级的参数。
		- **位置：**它位于配置文件的最顶层，不包含在任何块（如 `http`）内。
		- `user`：这个指令定义了 Nginx 工作进程应该以哪个用户和组的身份运行。
		  collapsed:: true
			- ```nginx
			  user  nginx;          # 使用 'nginx' 用户运行
			  # 或者更详细地：
			  # user  www-data staff;
			  ```
		- `worker_processes`：这个指令控制 Nginx 启动多少个工作进程来处理请求。
		  collapsed:: true
			- 当 Nginx 启动后，主进程会根据 `worker_processes` 启动相应数量的工作进程，所有工作进程都以 `user` 指定的身份运行，并从这个全局环境中继承参数。
			- ```nginx
			  worker_processes  4;  # 如果你的服务器有 4 个 CPU 核心
			  # 或者使用自动检测（推荐）：
			  worker_processes  auto;
			  ```
		- `error_log`：定义 Nginx 错误日志的位置和级别。
		  collapsed:: true
			- ```nginx
			  error_log  /var/log/nginx/error.log  notice; # 生产环境常用 notice 或 warn
			  ```
		- `pid`：定义主进程 ID 文件存放的位置。
		  collapsed:: true
			- Nginx 在启动后，会将主进程的 PID 写入此文件，这样你就可以使用此 PID 来向 Nginx 发送信号（如重新加载配置）。
			- ```nginx
			  pid        /var/run/nginx.pid;
			  ```
	- ### `events` 配置
		- **作用：**它定义了所有工作进程处理网络连接的行为。
		- `worker_connections`：设定单个工作进程可以同时维护的最大连接数（或并发连接数）。
		  collapsed:: true
			- ```nginx
			  worker_connections  1024;  # 单个工作进程最多能处理 1024 个连接
			  ```
	- ### `http` 配置
		- **作用：**所有的 Web 服务配置都必须放在 `http` 块内部。它定义了 Nginx 作为 HTTP 服务器时的设置。
		- `include /etc/nginx/mime.types;`：引入了 MIME 类型映射表，让 Nginx 知道如何设置 `Content-Type`。
		  collapsed:: true
			- 它告诉 Nginx 当返回文件时，应该在 HTTP 响应头中包含正确的 `Content-Type`。
			- 它将文件扩展名（如 `.html`, `.css`, `.jpg`）映射到对应的 MIME 类型（如 `text/html`, `text/css`, `image/jpeg`）。
		- `default_type`：如果 Nginx 找不到特定文件的 MIME 类型，它会使用这里定义的默认值。
		  collapsed:: true
			- ```nginx
			  default_type  application/octet-stream; # 表示作为通用二进制流下载
			  ```
		- `log_format`：定义了访问日志应该记录什么信息，以及这些信息的格式。
		  collapsed:: true
			- ```nginx
			  http {
			      # ...
			      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
			                        '$status $body_bytes_sent "$http_referer" '
			                        '"$http_user_agent" "$http_x_forwarded_for"';
			      # ...
			  }
			  ```
			- `$remote_addr`：客户端 IP
			- `$status`：HTTP 状态码
			- `$request_time`：总处理时间
			- `$upstream_response_time`：后端响应时间
		- `access_log`：定义访问日志放在哪，以及使用哪种格式。
		  collapsed:: true
			- ```nginx
			  access_log  /var/log/nginx/access.log  main;
			  ```
		- `keepalive_timeout`：客户端在一个连接上保持活动的最大时间。如果在这个时间内没有新的请求，连接会被关闭。用于减少客户端与服务器之间的 TCP 握手次数。
		- `sendfile`：启用后，Nginx 可以直接将文件数据从内核缓冲区发送到 Socket，绕过用户空间，极大地提高了发送静态文件的效率。
		- `server_tokens`：隐藏 Nginx 版本号，防止泄露版本信息给潜在攻击者。
		- `client_max_body_size`：设置请求体大小。限制客户端请求体（例如上传的文件）的最大尺寸。防止内存耗尽。
		- `proxy_read_timeout`：读取超时，Nginx 等待后端服务器返回数据的最长时间。防止后端卡死导致 Nginx 连接被长时间占用。
		- `include`：将另一个文件中的配置内容原封不动地插入到当前配置所在的位置。
		  collapsed:: true
			- 如果所有配置都堆在一个 5000 行的 `nginx.conf` 文件里，维护起来将是噩梦。通过 `include`，可以将不同职能的配置分开。
			- **注意：**
				- `include` 进来的内容必须符合它被放置的上下文。你不能在 `main` 块中 `include` 一个包含 `location` 块的文件（因为 `location` 只能在 `server` 块内）。
		- `upstream`：用于定义一组后端服务器（目标地址池和分配规则）。当 Nginx 作为反向代理时，它会将客户端请求根据负载均衡策略分发到这个 `upstream` 组中的某台服务器上。
		  collapsed:: true
			- ```nginx
			  http {
			      # ... 其他 http 配置 ...
			  
			      upstream backend_group_name {
			          # 负载均衡策略 (默认是轮询)
			          least_conn; # 或 ip_hash; 或其他
			  
			          # 后端服务器列表 (Server Definitions)
			          server server1.example.com:80 weight=3;
			          server 192.168.1.100:8080 backup;
			          server standby.example.com:8080 down;
			      }
			  
			      # ... server 块会使用 upstream_group_name ...
			  }
			  ```
			- `server`：指定后端服务器的 IP:端口 或 域名。
			- `weight=`：权重。分配给该服务器的请求比例。默认权重为 1。
			- `backup`：备用服务器。仅当该组中所有非 `backup` 服务器都不可用时，请求才会发送给它。
			- `down`：禁用/停用。永久标记该服务器不可用，请求不会发给它。
			- **负载均衡策略：**
				- **轮询（默认）**：没有指定任何负载均衡策略时使用的默认策略。每个请求按顺序循环发送到下一台服务器。
				- **最少连接：**`least_conn;` 将请求发送给当前活动连接数最少的服务器。
				- **IP 哈希**：`ip_hash;` 根据客户端的 IP 地址计算哈希值，确保来自同一 IP 的请求始终访问同一台后端服务器。用于需要会话保持（Sticky Session）的有状态服务。
				- **哈希：**`hash $request_uri;` 根据 Nginx 变量（如 URI、Header）计算哈希值，确保同一资源的请求访问同一台服务器。
		- `server`：定义了一个独立的网站或服务实例。
			- **作用：**
				- `server` 块的作用是判断请求属于哪个网站，然后将处理权交给对应的 `location` 块，由它决定该请求（URI）应如何处理——是返回静态文件、转发到上游服务，还是执行重定向。
				- 可以理解为在告诉 Nginx：当请求到达某个域名或端口时，应将其交给哪一类处理逻辑（`location`），并由哪个后端服务（`upstream`）来处理。这正是 `server` 块（虚拟主机）的职责。
				- 一个 Nginx 实例可以根据不同的 `listen` 和 `server_name` 指令，承载无数个独立的 `server` 块。
			- `listen`：监听的 IP 地址和端口。这是 Nginx 接收请求的入口。
			- `server_name`：域名匹配。当请求的 `Host` 头与此名称匹配时，使用此 `server` 块。
			- `add_header`：用于在 Nginx 向客户端发送 HTTP 响应时，添加自定义的响应头字段。
			  collapsed:: true
				- ```nginx
				  add_header name value [always];
				  add_header X-Download-Options "noopen" always;
				  ```
				- `always`（可选）：默认情况下，`add_header` 仅在状态码为 200、201、204、206、301、302、303、304、307、308 时生效。若要在错误页面（如 404、500）中也添加自定义响应头，需使用 `always`。
			- `error_page`：当 Nginx 遇到特定错误状态码时，应该如何响应客户端。
			  collapsed:: true
				- **语法：**`error_page code [ =response ] uri;`
					- **`code`**：要捕获的错误状态码（如 404、500、503）。
					- **`=response`**（可选）：指定在内部重定向到新 URI 后，返回给客户端的最终状态码；若省略，则返回原始错误码。
					- **`uri`**：内部重定向的目标位置，通常对应一个 `location` 块。
				- **示例：**
					- ```nginx
					  # 当 backend_app 返回 502 或 504 时，返回 503 状态码并显示静态页面
					      error_page 502 504 =503 /maintenance.html;
					  
					      location = /maintenance.html {
					          root /usr/share/nginx/html;
					          internal;
					      }
					  ```
					- `internal`：确保只有 Nginx 内部重定向能访问这个 URI
					- `root`：定义了静态资源的起点，用于指定一个目录路径，Nginx 在处理静态文件请求时，会从这个目录开始查找文件。
						- 当 Nginx 收到请求时，会根据以下规则构建文件路径：最终文件路径=root 路径+请求的 URI
			- `location`：定义了 Nginx 应该对匹配到 URI 的所有请求执行什么操作。
				- **修饰符：**
					- | **Modifier** | **名称** | **匹配行为** | **优先级** |
					  | ---- | ---- | ---- |
					  | **`=`** | 精确匹配 (Exact Match) | 匹配 URI 必须完全一致，且匹配成功后立即停止搜索。 | 最高 (Step 1) |
					  | **`^~`** | 前缀停止匹配 (Prefix Stop) | 执行最长前缀匹配。如果匹配成功，Nginx 将停止搜索，跳过正则表达式评估。 | 高 (Step 3) |
					  | **`~`** | 区分大小写的正则匹配 (Case-sensitive Regex) | 使用区分大小写的正则表达式进行匹配。按配置顺序评估。 | 中 (Step 4) |
					  | **`~*`** | 不区分大小写的正则匹配 (Case-insensitive Regex) | 使用不区分大小写的正则表达式进行匹配。按配置顺序评估。 | 中 (Step 4) |
					  | **(None)** | 前缀匹配 (Prefix Match) | 执行最长前缀匹配。但搜索会继续到正则匹配阶段。 | 最低 (Step 2/5) |
				- `proxy_pass`：它负责将客户端的请求转发到指定的后端服务器（Upstream）。
					- `proxy_pass` 是否带斜杠会影响请求 URI 的拼接方式：
						- 带斜杠：会去掉 `location` 匹配的部分，用新路径替换。
							- ```nginx
							  location /api/ { proxy_pass http://backend/; }
							  # /api/users → http://backend/users
							  ```
						- 不带斜杠：会保留 `location` 匹配的部分，直接追加到后端地址后。
							- ```nginx
							  location /api/ { proxy_pass http://backend; }
							  # /api/users → http://backend/api/users
							  ```
- ## Nginx 的原理
	- **转发请求的流程**
	  collapsed:: true
		- 当 Nginx 接收到与某个 `location` 匹配的请求时，会触发 `proxy_pass`，并执行以下操作：
			- **连接后端：** 建立与 `proxy_pass` 指定的后端服务器的连接。
			- **发送请求：** 将客户端的请求（包含请求头、请求体等）转发给后端服务器。
			- **接收响应：** 获取后端服务器返回的响应数据。
			- **返回客户端：** 将响应内容（可能经过 Nginx 处理或修改）再返回给客户端。
-