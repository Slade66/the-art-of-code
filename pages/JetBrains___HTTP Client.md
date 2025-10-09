-
- **文件变量**
	- 文件变量仅在当前 `.http` 文件内生效。
	- **如何定义和使用：**
		- 在 `.http` 文件顶部使用 `@` 符号定义变量，在后续请求中通过 `{{}}` 语法引用变量。
		- ```http
		  # 文件变量定义
		  @hostname = localhost
		  @port = 8000
		  @baseUrl = {{hostname}}:{{port}}
		  
		  ###
		  
		  # 使用文件变量
		  GET http://{{baseUrl}}/v1/health
		  ```
-