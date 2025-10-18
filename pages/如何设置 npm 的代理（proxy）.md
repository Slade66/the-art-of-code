- **查看当前的代理配置：**
	- ```bash
	  npm config get proxy
	  npm config get https-proxy
	  ```
- **移除代理配置：**
	- ```bash
	  npm config rm proxy
	  npm config rm https-proxy
	  ```
- **启动代理：**
	- 确保你的代理软件已经**开启**，并且正在 `127.0.0.1:XXX` 这个地址和端口上运行。
	- 例如，如果你的代理端口是 `7897`：
		- ```bash
		  npm config set proxy http://127.0.0.1:7897
		  npm config set https-proxy http://127.0.0.1:7897
		  ```
-