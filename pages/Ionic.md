-
- ## CLI
	- 安装：`npm install -g @ionic/cli`
	- 创建项目：`ionic start`
	- 运行项目：`ionic serve`
	- **Android：**
		- 安装软件包：`npm install @capacitor/android`
		- 添加 Android 平台：`npx cap add android`
		- 运行项目：`npx cap run android`
			- **Gradle 报错问题的解决方案：**
				- **下载 Gradle 出错**
				  collapsed:: true
					- **报错现象：**
						- ```
						  × Running Gradle build - failed!
						  [error] Downloading https://services.gradle.org/distributions/gradle-8.11.1-all.zip
						          
						          Exception in thread "main" java.net.ConnectException: Connection refused: no further information
						          at java.base/sun.nio.ch.Net.pollConnect(Native Method)
						          at java.base/sun.nio.ch.Net.pollConnectNow(Net.java:672)
						          at java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)
						          at java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)
						          at java.base/java.net.Socket.connect(Socket.java:633)
						          at java.base/sun.net.NetworkClient.doConnect(NetworkClient.java:178)
						          at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:534)
						          at java.base/sun.net.www.http.HttpClient$1.run(HttpClient.java:593)
						          at java.base/sun.net.www.http.HttpClient$1.run(HttpClient.java:591)
						          at java.base/java.security.AccessController.doPrivileged(AccessController.java:569)
						          at java.base/sun.net.www.http.HttpClient.privilegedOpenServer(HttpClient.java:590)
						          at java.base/sun.net.www.http.HttpClient.openServer(HttpClient.java:634)
						          at java.base/sun.net.www.protocol.https.HttpsClient.<init>(HttpsClient.java:266)
						          at java.base/sun.net.www.protocol.https.HttpsClient.New(HttpsClient.java:380)
						          at
						          java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.getNewHttpClient(AbstractDelegateHttpsURLConnection.java:202)
						          at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect0(HttpURLConnection.java:1262)
						          at java.base/sun.net.www.protocol.http.HttpURLConnection.plainConnect(HttpURLConnection.java:1127)
						          at
						          java.base/sun.net.www.protocol.https.AbstractDelegateHttpsURLConnection.connect(AbstractDelegateHttpsURLConnection.java:179)
						          at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1686)
						          at java.base/sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1610)
						          at java.base/sun.net.www.protocol.https.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:224)
						          at org.gradle.wrapper.Install.forceFetch(SourceFile:2)
						          at org.gradle.wrapper.Install$1.call(SourceFile:8)
						          at org.gradle.wrapper.GradleWrapperMain.main(SourceFile:67)
						  
						  ```
					- **解决步骤：**
						- **手动下载 Gradle 包：**点击链接下载 `gradle-8.11.1-all.zip`（注意：请勿解压）。
						  logseq.order-list-type:: number
						- **放置文件：**将下载好的 zip 文件直接复制到项目的 `android/gradle/wrapper/` 目录下。
						  logseq.order-list-type:: number
						- **修改配置：**打开同目录下的 `gradle-wrapper.properties` 文件，找到 `distributionUrl` 字段，去除路径中的 URL 前缀，仅保留文件名。
						  logseq.order-list-type:: number
							- 修改后示例：`distributionUrl=gradle-8.11.1-all.zip`
				- **网络连接报错**
				  collapsed:: true
					- **报错现象：**`Connect to 127.0.0.1:10809 [/127.0.0.1] failed: Connection refused: no further information`
					- **问题原因：**Gradle 的全局代理端口设置错误（例如配置了错误的端口号 10809）。
					- **解决步骤：**
						- 进入用户主目录，打开 `.gradle/gradle.properties` 文件。
						  logseq.order-list-type:: number
						- 修改端口配置：
						  logseq.order-list-type:: number
							- ```
							  systemProp.http.proxyHost=127.0.0.1
							  systemProp.http.proxyPort=7897
							  
							  systemProp.https.proxyHost=127.0.0.1
							  systemProp.https.proxyPort=7897
							  ```
				- **找不到 Android SDK**
				  collapsed:: true
					- **报错现象：**
						- ```
						  > SDK location not found. Define a valid SDK location with an ANDROID_HOME environment variable or by setting
						  the sdk.dir path in your project's local properties file at
						  'A:\Users\liyuz\Desktop\zry\android\local.properties'.
						  ```
					- **问题原因：**Android 项目里缺少一个名为 `local.properties` 的配置文件。
					- **解决步骤：**
						- 在 android 项目下新建一个文件：`local.properties`
						  logseq.order-list-type:: number
						- 在里面写入：
						  logseq.order-list-type:: number
							- ```
							  ## This file must *NOT* be checked into Version Control Systems,
							  # as it contains information specific to your local configuration.
							  #
							  # Location of the SDK. This is only used by Gradle.
							  # For customization when using a Version Control System, please read the
							  # header note.
							  #Sat Nov 22 10:15:21 CST 2025
							  sdk.dir=A\:\\Users\\liyuz\\AppData\\Local\\Android\\Sdk
							  
							  ```
							- **SDK 在哪？**打开 Android Studio -> 启动界面点击 "左下角的小齿轮" -> "Settings" -> "Android SDK" -> 顶部有个 "Android SDK Location"，把那个路径复制下来，把里面的 `\` 改成 `\\` 填进去。
				- **Java 版本错误**
				  collapsed:: true
					- **错误现象：**
						- ```
						  Execution failed for task ':capacitor-android:compileDebugJavaWithJavac'.  
						  > error: invalid source release: 21
						  ```
					- **问题原因：**Capacitor 7 默认要求使用 Java 21 编译，但当前环境 JDK 版本低于 17 或更低。
					- **解决步骤：**
						- 下载并安装 JDK 21
						  logseq.order-list-type:: number
						- 设置环境变量：
						  logseq.order-list-type:: number
							- 新建或修改系统环境变量 `JAVA_HOME`，值为 JDK 21 的安装根目录。
							  logseq.order-list-type:: number
							- 编辑系统 Path 变量，在最顶部新增 %JAVA_HOME%\bin
							  logseq.order-list-type:: number
			- **把项目运行到真机上：**
			  collapsed:: true
				- 拿出一台 Android 手机，用数据线连上电脑。
				- 手机必须开启 “开发者模式” 和 “USB 调试”。
		- 在 Android Studio 中打开 Android 项目：`npx cap open android`
-