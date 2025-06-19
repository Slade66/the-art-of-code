有什么用？
heading:: true
	- **流量控制：**
		- 它能根据你设定的规则，对到达服务的请求流量进行精确地控制，防止因突发流量冲垮服务。
		- **控制维度：**
			- **QPS（每秒请求数）：**这是最常用的模式。你可以规定某个接口（例如 `GET /users/{id}`）每秒最多只能被调用 100 次。超过这个阈值的请求会被立即拒绝。
			- **并发线程数：**你可以规定某个接口在同一时刻，最多只能有 10 个线程在处理它。这能有效防止因慢调用导致线程堆积。
	- **熔断降级：**
		- 这是 Sentinel “高灵敏保险丝”的角色。它会持续监控对下游服务（或其他不稳定资源）的调用情况，当发现这个依赖变得不稳定时，会自动“拉闸熔断”，在一段时间内阻止所有对该服务的调用，从而保护自身服务不被拖垮。
		- **熔断策略（“拉闸”的依据）：**
			- **慢调用比例：**当对某个服务的请求中，响应时间超过某个阈值（比如 200ms）的请求比例，在一段时间内超过了你设定的百分比，就会触发熔断。
			- **异常比例 / 异常数：**当对某个服务的调用，在一段时间内失败的比例或失败的总数超过了阈值，就会触发熔断。
	- **系统自适应保护：**
		- Sentinel 会监控当前服务实例的总负载情况（如 `CPU Usage` 等）。当系统负载超过了预设的阈值，说明系统即将达到极限，此时 Sentinel 会主动拒绝一部分入口请求，以防止整个服务因资源耗尽而崩溃。
	- **Sentinel 控制台（Dashboard）：**
		- **实时监控：**以秒级的精度，实时监控所有微服务的接口调用情况（QPS、响应时间、拒绝数等）。
		- **动态规则管理：**无需重启服务，可以直接在控制台界面上为任何接口动态地添加、修改或删除流量控制和熔断降级规则，规则会实时生效。
- 怎么用？
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