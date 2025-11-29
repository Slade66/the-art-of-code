- **作用：**
	- `server` 块定义了一个独立的网站或服务实例。作用是判断请求属于哪个网站，然后将处理权交给对应的 `location` 块，由它决定该请求（URI）应如何处理——是返回静态文件、转发到上游服务，还是执行重定向。
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
- `location`：它决定了请求 URI 如何匹配到特定的配置块，进而决定应该对匹配到 URI 的请求执行什么处理。
	- **匹配符：**
		- `=`：精确匹配。优先级最高。匹配 URI 必须完全一致，且匹配成功后立即停止搜索。
		- `^~`：前缀停止匹配。优先级次高。执行最长前缀匹配，如果匹配上，就停止搜索，不再去匹配正则表达式。
		  collapsed:: true
			- **例子：**
				- 用户访问了这个地址：`http://example.com/static/js/main.js`
				- 没有 `^~` 时：
					- ```nginx
					  location /static/ {
					      root /var/www;
					  }
					  
					  location ~ \.php$ {
					      fastcgi_pass 127.0.0.1:9000;
					  }
					  
					  ```
				- Nginx 会：
					- 先找到 `/static/` 这个前缀匹配；
					- 但因为它没有 `^~` 修饰符，Nginx 还会继续去检查所有正则表达式（比如 `~ \.php$`）；
					- 发现正则没匹配成功（因为 `main.js` 不是 `.php`），于是才回退到 `/static/`。
				- 虽然最终结果没错，但这中间多了一次不必要的“正则匹配运算”，结果白忙活一圈。
				- 如果带了 `^~`，匹配到 `/static/` 前缀，立刻确定使用这个块，完全跳过正则表达式匹配。
		- `~`：区分大小写的正则匹配，按配置顺序评估。优先级中。
		- `~*`：不区分大小写的正则匹配。按配置顺序评估。优先级中。
		- **无修饰符，直接写路径：**前缀匹配。执行最长前缀匹配。只要请求路径以 `/xxx` 开头，就算匹配成功。但搜索会继续到正则匹配阶段。优先级低。
			- `location / {...}`
				- 它匹配任何以 `/` 开头的请求 URI。
				- 这是一个兜底配置，因为所有的 URL 都是以 `/` 开头的（例如 `/index.html`, `/api/user`, `/images/logo.png`），所以，当所有其他更具体的 `location` 块都未能命中时，Nginx 就会自动落到这个 `location /` 块来处理请求。
		- **注意：**
			- 如果将一个范围宽泛的正则匹配放在前面，它可能会阻止后续更精确的匹配生效。
			- 正则表达式的计算通常比简单的前缀匹配消耗更多的 CPU 资源。
	- **匹配流程：**
		- **精确匹配：**Nginx 首先检查所有带有 `=` 修饰符的 `location` 块。若请求 URI 与其中某个完全匹配，Nginx 会立即停止后续查找并使用该块。
		- **最长前缀匹配：**若未找到精确匹配，Nginx 会遍历所有非正则表达式的 `location`（即无修饰符或带 `^~` 的块），找出与请求 URI 最长的匹配前缀，并将其记录为潜在的备用结果。
		- **前缀停止符：**接着，Nginx 检查上一步中找到的最长前缀是否带有 `^~` 修饰符。若是，则立即选用该块并跳过正则匹配阶段。
		- **正则匹配：**只有当上一步未触发停止条件时，Nginx 才会依次按配置文件中的顺序，评估所有 `~` 和 `~*` 的正则表达式 `location`。第一个成功匹配请求 URI 的块将被选用，并终止搜索。
		- **回退：**如果所有正则表达式匹配均失败，Nginx 将使用 Step 2 记录的最长前缀匹配块处理请求。若连前缀匹配也不存在，则最终使用 `location /` 作为默认的兜底规则。
	- `proxy_pass`：它负责将客户端的请求转发到指定的后端服务器（Upstream）。
	  collapsed:: true
		- `proxy_pass` 是否带斜杠会影响请求 URI 的拼接方式：
			- 带斜杠：会去掉 `location` 匹配的部分，剩下的部分拼到上游地址后面。
				- ```nginx
				  location /api/ { proxy_pass http://backend/; }
				  # /api/users → http://backend/users
				  ```
			- 不带斜杠：会保留 `location` 匹配的部分，直接追加到后端地址后。
				- ```nginx
				  location /api/ { proxy_pass http://backend; }
				  # /api/users → http://backend/api/users
				  ```
	- `proxy_buffering`：用于控制 Nginx 是否缓存上游服务器（被代理服务）的响应数据。
	  collapsed:: true
		- **开启缓冲时**，Nginx 会先将上游返回的数据读入内存（或临时文件），待积累到一定量后再转发给客户端。可以理解为服务员先把菜全部端到自己手边，再慢慢端给客人。
		- **关闭缓冲时**，Nginx 会实时转发上游返回的数据，一边接收一边传给客户端，不会将其存入内存或临时文件。就像服务员从厨师手里接过菜后立即送到客人手中，不作等待。
	- `proxy_set_header`：用于在 Nginx 将客户端请求转发给上游服务器时，设置或修改要发送的请求头。
		- 它通常用于传递客户端的真实信息（如 IP、Host、协议等），或为特定功能（如 WebSocket 升级、身份验证）添加必要的头字段，使上游服务能够正确识别请求来源和上下文。
		- 语法：`proxy_set_header <Header-Name> <Value>;`
		- `$remote_addr`：客户端的 IP 地址。
		- `$host`：请求头里的 `Host:` 字段，或收到请求的 IP 地址。
		  collapsed:: true
			- **作用：**
				- 告诉服务器，我是通过哪个域名来访问你的。
				- 同一台服务器可以托管很多网站，就像一栋大楼里有很多公司，Host 头就是写在信封上的公司名称（“我要找哪家公司”），没有 Host 头，服务器根本不知道你是要找哪家公司（访问哪一个网站）。
			- 域名访问：
				- 用户访问：`https://a.example.com/hello`
				- 浏览器自动带上：`Host: a.example.com`
				- 则：`$host = "a.example.com"`
			- IP 直连：
				- 用户访问：`http://1.2.3.4/hello`
				- 浏览器发出的头：`Host: 1.2.3.4`
				- 所有：`$host = "1.2.3.4"`
		- `$proxy_add_x_forwarded_for`：
		  collapsed:: true
			- 它会设置 `X-Forwarded-For` 这个 HTTP 头部，帮助后端服务器识别出最初发起请求的客户端真实 IP 地址，而不是 Nginx 代理的 IP。
			- 当 Nginx 作为反向代理（Reverse Proxy）时，它会接收客户端的请求，然后转发给后端的应用服务器。如果没有特殊处理，后端服务器看到的请求 IP 永远是Nginx 服务器的 IP 地址，而不是客户端的真实 IP。
			- **工作原理：**
				- 它会自动获取当前请求的源 IP（即 `$remote_addr`，也就是客户端或上一个代理的 IP）。
				- 它会将这个 IP 地址追加到现有的 `X-Forwarded-For` 请求头部的末尾。如果请求中本来就没有这个头部，它就会创建一个。
			- **例子：**
				- 如果客户端 IP 是 `103.45.67.89`，请求依次经过代理 A (`192.168.1.1`) 和代理 B (Nginx)，那么后端服务器收到的 `X-Forwarded-For` 头部就会是：
					- ```
					  X-Forwarded-For: 103.45.67.89, 192.168.1.1
					  ```
	- `root`：它将客户端请求的 URI 完整地追加到 root 定义的路径后面，形成最终要查找的文件系统路径。指定了处理此请求时，Nginx 应该去查找文件的本地文件系统根路径。
	  collapsed:: true
		- **例子：**
			- ```nginx
			  location / { # 兜底王
			    root /usr/share/nginx/html;
			  }
			  ```
			- 请求 `/index.html` 时，路径匹配到 `/`，Nginx 随后从 `/usr/share/nginx/html/index.html` 读取对应的文件。
	- `index`：定义了 Nginx 在处理一个以斜杠 `/` 结尾的请求（即请求一个目录而不是文件）时，应该尝试查找的默认文件名。当请求指向一个目录时，应该用哪个文件来响应。
	  collapsed:: true
		- `index` 指令可以接受一个或多个文件名。Nginx 会按照它们在指令中出现的顺序，在请求目录中逐一查找。
		- **内部重定向机制：**
			- 当 Nginx 成功通过 `index` 找到一个文件时，它不会直接发送这个文件。相反，它会默默地执行一个内部重定向：
				- 如果请求 `/blog/`，Nginx 找到了 `/blog/index.html`。
				- Nginx 会在内部将请求重写为 `/blog/index.html`。
				- 这个新请求 (`/blog/index.html`) 会重新进入 `location` 匹配流程。
			- 巧妙地将目录请求导向到合适的文件处理 `location` 块，实现网站默认主页功能。
			- **例子：**
				- ```nginx
				  location / {
				      root /var/www/html;
				      index index.php index.html;
				  }
				  
				  location ~ \.php$ {
				      # ... PHP FastCGI 配置，专门处理 .php 文件
				  }
				  ```
				- 如果请求 `/`，Nginx 找到 `index.php`，然后将请求内部重定向到 `/index.php`。这个新请求 `/index.php` 就会成功命中第二个 `location` 块，从而被 PHP 解释器处理。完美！
	- `try_files`：指示 Nginx 按顺序尝试访问一系列指定的文件或 URI，一旦找到可用项就停止继续查找；若全部尝试失败，则执行最后一个参数指定的操作。
	  collapsed:: true
		- `$uri` 与 `$uri/` 的区别：`$uri` 用于查找文件，而 `$uri/` 用于查找目录，并会触发 `index` 指令在该目录下继续查找指定文件。
		- 当最后一个参数是 URI（如 `/index.html` 或 `@named_location`）时，Nginx 会执行一次内部重定向，新的 URI 将重新进入 `location` 匹配流程。
		- 如果最后一个参数是 `=code`（如 `=404` 或 `=500`），Nginx 会直接返回对应的 HTTP 状态码，并终止后续处理。
		- **例子：**
			- ```nginx
			  location / {
			      root   /app/web;
			      index  index.html;
			      try_files $uri $uri/ /index.html;
			  }
			  ```
			- `try_files` 会按顺序（从左到右）检查文件是否存在：
				- `$uri`：尝试按字面意思查找文件。
				- `$uri/`：如果上一步找不到，尝试把它当成一个目录（并结合 `index` 指令）。
				- `/index.html`：如果前两步都失败了，就内部重定向到 `/index.html`。
					- Nginx 重新开始处理 `/index.html`，这个请求会被同一个 `location /` 块捕获，并最终返回 `/app/web/index.html` 的内容。