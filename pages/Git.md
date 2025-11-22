- [[CRLF 和 LF 混用导致的“虚假改动”问题]]
- [[分支（Branch）]]
- ## Git 配置代理
  collapsed:: true
	- ```bash
	  # 查看当前 Git 代理是什么
	  git config --global --get http.proxy
	  git config --global --get https.proxy
	  
	  # 设置 Git 使用 7897 端口
	  git config --global http.proxy http://127.0.0.1:7897
	  git config --global https.proxy http://127.0.0.1:7897
	  ```
-