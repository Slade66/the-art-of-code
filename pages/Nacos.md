有什么用？
heading:: true
	- **服务注册与发现：**
		- 在微服务架构中，Nacos 充当“服务通讯录”的角色。
		- 服务实例启动后会将自身的地址信息（如 IP 和端口）注册到 Nacos；当某个服务需要调用其他服务时，会先向 Nacos 查询目标服务的地址列表，并通过负载均衡选择一个可用实例发起调用。
		- 借助这一机制，系统实现了服务地址的动态管理，避免硬编码，提升了服务间通信的灵活性与可扩展性。
	- **分布式配置管理：**
		- 随着微服务数量的不断增加，各服务通常拥有各自独立的配置文件，配置内容不仅重复，而且分散，导致管理过程既繁琐又容易出错。
		- **集中管理：**Nacos 提供集中式配置管理平台，实现配置的统一维护与共享。各服务在启动时可自动从 Nacos 拉取所需配置，省去了手动维护多个配置文件的繁琐操作。
		- **动态刷新：**Nacos 同时支持配置的热更新，服务在配置变更后可实时感知并应用新配置，无需重启，从而显著提升系统的灵活性与运维效率。
- 怎么用？
  heading:: true
	- **服务注册与发现：**
		- **搭建 Nacos Server：**
			- ```bash
			  docker run -d \
			  --name nacos-standalone \
			  -p 8848:8848 \
			  -e MODE=standalone \
			  nacos/nacos-server
			  ```
		- **添加依赖：**
			- ```xml
			  <dependency>
			      <groupId>com.alibaba.cloud</groupId>
			      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
			  </dependency>
			  ```
		- **配置 `application.yml`：**
			- ```yml
			  spring:
			    application:
			      name: user-service   # 当前服务的名称，会作为注册到 Nacos 的服务名
			    cloud:
			      nacos:
			        discovery:
			          server-addr: 127.0.0.1:8848   # Nacos 服务地址
			  ```
		- **在你的 Spring Boot 启动类上添加 `@EnableDiscoveryClient` 注解：**
			- ```java
			  import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
			  import org.springframework.boot.SpringApplication;
			  import org.springframework.boot.autoconfigure.SpringBootApplication;
			  
			  @SpringBootApplication
			  @EnableDiscoveryClient
			  public class UserServiceApplication {
			      public static void main(String[] args) {
			          SpringApplication.run(UserServiceApplication.class, args);
			      }
			  }
			  ```
	- **分布式配置管理：**
		- 几乎所有你原本写在本地配置文件中的内容，都可以，也应当放到 Nacos 上进行管理。
		- **在 Nacos 控制台发布配置：**
			- 在左侧导航栏选择 “配置管理” -> “配置列表”，点击“创建配置”。
			- 填写配置信息：
				- **Data ID：**配置的唯一标识符，需与 `bootstrap.yml` 中的对应项保持一致。
				- **Group：**配置分组，用于区分不同的项目或运行环境。
				- **配置格式：**`YAML`
				- **配置内容：**填写 `application.yml` 中的内容，支持使用变量占位。若本地存在同名配置，本地优先。
			- 点击“发布”完成配置创建。
		- **引入依赖：**
			- ```xml
			  <dependency>
			      <groupId>com.alibaba.cloud</groupId>
			      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
			  </dependency>
			  <dependency>
			  	<groupId>org.springframework.cloud</groupId>
			  	<artifactId>spring-cloud-starter-bootstrap</artifactId>
			  </dependency>
			  ```
			- `spring-cloud-starter-bootstrap`：用于在 Spring 应用启动的早期阶段加载 `bootstrap.yml`，以便优先从外部配置源（如 Nacos）获取配置，早于 `application.yml` 生效。
			- Spring Cloud 2021.x 及以后版本引入了新的配置加载机制，默认支持加载远程配置，无需再依赖 `bootstrap.yml` 和相关依赖。
		- **配置 `bootstrap.yml`：**
			- 在 `src/main/resources` 下创建 `bootstrap.yml` 文件。
			- `bootstrap.yml` 的加载优先级高于 `application.yml`。应用启动时会先读取 `bootstrap.yml`，根据其中的配置从 Nacos 拉取外部配置，并与本地的 `application.yml` 合并后，完成初始化启动。
			- ```yaml
			  spring:
			    application:
			      # 这是你的服务名，也会作为 Nacos 配置的 Data ID 的一部分
			      name: user-service
			    cloud:
			      nacos:
			        config:
			          # Nacos Server 的地址
			          server-addr: 127.0.0.1:8848
			          # 配置文件的后缀名，如 yml, properties
			          file-extension: yml
			          # 配置所属的分组，默认为 DEFAULT_GROUP
			          # group: DEV_GROUP 
			          # 配置所属的命名空间，用于环境隔离
			          # namespace: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
			  ```
		- **在代码中使用配置并开启动态刷新：**
			- **`@Value` + `@RefreshScope`：**
				- ```java
				  import org.springframework.beans.factory.annotation.Value;
				  import org.springframework.cloud.context.config.annotation.RefreshScope;
				  import org.springframework.web.bind.annotation.GetMapping;
				  import org.springframework.web.bind.annotation.RestController;
				  
				  @RestController
				  // @RefreshScope 注解是实现配置动态刷新的关键！
				  // 它会使得这个 Bean 在接收到配置变更事件时，被销毁并重新创建一个新的实例，从而加载到最新的配置。
				  @RefreshScope 
				  public class ConfigController {
				  
				      // 使用 @Value 注解注入配置
				      @Value("${user.level.default}")
				      private String userDefaultLevel;
				      
				      // 你甚至可以给一个默认值，以防 Nacos 上没有这个配置
				      @Value("${some.other.config:defaultValue}")
				      private String someConfig;
				  
				      @GetMapping("/level")
				      public String getUserLevel() {
				          return "Current default user level is: " + userDefaultLevel;
				      }
				  }
				  ```
			- **`@ConfigurationProperties`：**
				- 这是 Spring Boot 中处理配置的最佳实践。它不是一个个地注入值（`@Value`），而是将一组相关的配置项映射到一个 Java 对象（POJO）上。
				- ```java
				  import lombok.Data; // 引入 Lombok 的 @Data 注解
				  import org.springframework.boot.context.properties.ConfigurationProperties;
				  import org.springframework.cloud.context.config.annotation.RefreshScope;
				  import org.springframework.stereotype.Component;
				  
				  @Component
				  @ConfigurationProperties(prefix = "user") // // 将 yml 中 "user" 前缀下的所有属性映射到这个类的字段上
				  @RefreshScope
				  @Data // 使用 @Data 注解替代所有手写的 getter/setter
				  public class UserProperties {
				  
				      private String name;
				      private String version;
				      private Address address;
				  
				      // 嵌套类同样可以使用 @Data 注解来简化
				      @Data
				      public static class Address {
				          private String province;
				          private String city;
				      }
				  }
				  ```
-