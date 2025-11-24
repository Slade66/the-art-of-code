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
- ## 项目结构
	- src
		- components
			- 存放自定义组件。组件的作用就是封装一段代码，方便在别的地方重复使用。
			- 当你发现有一段界面代码在好几个页面都要用时，就把它切出来放到这。
			-
		- router
			- `index.ts`：它是整个应用的地图。决定了“用户访问什么网址，显示哪个页面”。当你需要加新页面，或者修改页面跳转关系时才打开它。
		- `views`
			- View = 视图 = 页面。
			- 这里放的文件，每一个通常都对应手机屏幕上的**一个完整页面**（全屏的）。
			- **对比**：`components` 目录放的是“小零件”（如一个按钮、一张卡片），而 `views` 放的是把这些零件拼好的“整个房间”。
	- `main.ts`：程序的入口，只要配置一次（比如安装了新插件）就不太动了。
	- `App.vue`：根组件，通常里面只有 `<ion-router-outlet>`，完全不用管。
	- `theme/variables.css`：除非你要改全局主题色（把蓝色变成红色），否则不用碰。
- ## 组件
	- `<ion-router-outlet>`
	  collapsed:: true
		- **作用：**提供一个视图容器，用于渲染您的 Vue 视图（在 Ionic 中，这些视图通常被称为 “页面”）。
		- **理解：**
			- 您可以将 `IonRouterOutlet` 视为一个舞台：
				- 当您导航到新页面时，新页面 **“上台”** (渲染)。
				- 当您返回上一页时，旧页面并不会立即 **“退场”** (销毁)，而是 **“留在后台”** (仍然在 DOM 中保留，但被隐藏)。
				- 当您再次切换回该页面时，它会**立刻出现**，**保留了它最后的状态**（就像演员只是在侧台休息，随时可以回到舞台的相同位置），这样就避免了重新加载和重建状态的延迟。
	- `<ion-tab-bar>`
	- `<ion-page>`
	  collapsed:: true
		- 是 Ionic 应用中用来包装每个视图（或“页面”）的顶层容器。
		- 因为 Ionic 是单页应用（SPA），切换页面时，本质上就是移除旧的 `<ion-page>`，再替换成新的。
		- 任何通过 **路由** (router) 导航到的视图**都必须**包含一个 `IonPage` 组件作为其根元素。
		- `IonPage` 赋予了您的视图一个**“页面身份”**，让它能参与到 Ionic 完整的导航和动画系统中。
		- **定义范围：** 当您创建一个新的 Vue 路由页面时，您将所有内容（`ion-header`、`ion-content` 等）都放在这个 **`IonPage` 容器内**。
		- **对接系统：** 这个相框（`IonPage`）有一个**特殊的“挂钩”**，能与上一组件 `IonRouterOutlet`（页面舞台）**完美对接**。
	- **`ion-header`** / **`ion-toolbar`** / **`ion-title`**: 顶部导航栏三件套。
	- `<ion-content>`
		- 所有需要滚动的内容（文章、列表、图片）都必须放在这里。如果把内容直接写在 `<ion-page>` 中而不使用 `<ion-content>`，页面将无法正常滚动，还可能被顶部导航栏遮挡。
-