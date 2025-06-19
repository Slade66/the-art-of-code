怎么用？
heading:: true
	- **第一步：启动 Sentinel 控制台**
		- 控制台是一个独立运行的 Java 程序，是一个可视化的监控和规则管理中心。
		- **下载 Dashboard：**
			- 前往 Sentinel 的官方 GitHub Releases 页面：https://github.com/alibaba/Sentinel/releases
			- 下载最新版本的 `sentinel-dashboard-x.x.x.jar` 文件。
		- **运行 Dashboard：**
			- 打开你的命令行/终端，进入 `.jar` 文件所在的目录。
			- 使用以下 `java -jar` 命令来启动它：
				- ```bash
				  java -jar sentinel-dashboard-1.8.8.jar
				  ```
				- 这条命令会以默认配置在 `8080` 端口启动控制台。如果你的 `8080` 端口被占用了，可以用 `-Dserver.port` 参数指定一个新端口，例如：`java -Dserver.port=8888 -jar ...`
		- **访问 Dashboard：**
			- 打开浏览器，访问 `http://localhost:8080`（或你指定的端口）。
			- 默认的登录用户名和密码都是 `sentinel`。
	- **第二步：在微服务中集成 Sentinel 客户端**
		- 让你的服务接入 Sentinel，受到 Sentinel 的保护，并与控制台通信。
		- **添加依赖：**
			- ```xml
			  <dependency>
			  	<groupId>com.alibaba.cloud</groupId>
			  	<artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
			  </dependency>
			  ```
		- **修改配置文件：**
			- 告诉你的微服务，Sentinel 在哪里。
			- ```yml
			  spring:
			    cloud:
			      sentinel:
			        # Sentinel 控制台 (Dashboard) 的地址
			        transport:
			          dashboard: localhost:8080
			          # 你的微服务会在本地启动一个API端口与Dashboard进行心跳通信，默认是8719
			          # 如果8719端口冲突，可以在这里修改
			          port: 8719 
			  ```
		- **启动你的微服务：**
			- 启动服务后，随意访问任意一个 API 接口（这一步非常重要，因为 Sentinel 采用懒加载机制，只有接口被实际访问后，才会将其作为资源上报至控制台）。
			- 然后刷新 Sentinel 控制台页面，你应该能在左侧菜单中看到你的微服务名称，这表明服务已成功接入。
			- 如果仍未显示，可能是由于 `spring-cloud-starter-alibaba-sentinel` 与当前 `spring-cloud` 版本不兼容所致。
	- **第三步：使用注解实现“熔断降级”**
		- 当我们的服务调用其它不稳定的服务时，需要用到熔断降级来保护自己。
		- ```java
		  @Service
		  public class UserService {
		  
		      /**
		       * 原始的业务方法
		       * @SentinelResource 用来标记这是一个受Sentinel保护的资源
		       * - value: 自定义的资源名称，必须唯一。
		       * - fallback: 指定降级方法的方法名。当此方法出现任何异常时，会调用降级方法。
		       */
		      @SentinelResource(value = "queryUserById", fallback = "queryByIdFallback")
		      public User queryById(Long id) {
		          // 模拟业务逻辑出现异常
		          if (id % 2 == 0) {
		              throw new RuntimeException("模拟查询用户时出现异常");
		          }
		          // 正常逻辑...
		          return new User(id, "用户" + id);
		      }
		  
		      /**
		       * 降级方法 (Fallback)
		       * - 方法签名必须和原方法完全一致。
		       * - 可以额外增加一个 Throwable 类型的参数，用来接收原始方法抛出的异常。
		       * @param id 原始方法的参数
		       * @param t  原始方法抛出的异常
		       * @return 降级处理后返回的数据
		       */
		      public User queryByIdFallback(Long id, Throwable t) {
		          System.out.println("进入了降级逻辑，异常信息：" + t.getMessage());
		          // 返回一个默认的“兜底”数据
		          return new User(id, "未知用户（因系统繁忙）");
		      }
		  }
		  ```
	- **第四步：使用控制台实现“流量控制”**
		- 在 Sentinel 控制台中选择服务资源，添加流控规则并设置限流阈值，保存后即可下发到客户端生效。
-