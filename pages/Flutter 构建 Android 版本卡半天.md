- **解决方法：**
	- 无需修改项目配置。
	- 在 `.gradle.properties` 中设置代理（此处以 v2ray 为例）：
		- ```properties
		  systemProp.http.proxyHost=127.0.0.1
		  systemProp.http.proxyPort=10809
		  
		  systemProp.https.proxyHost=127.0.0.1
		  systemProp.https.proxyPort=10809
		  ```
	- 使用手机热点，不要连接学校网络。
-