怎么用？
heading:: true
	- **第一步：创建独立的网关项目**
		- “首先，需要单独创建一个全新的 Spring Boot 项目，因为网关本身就是一个独立的微服务。
	- **第二步：引入核心依赖**
		- 在你的网关项目的 `pom.xml` 文件中，你需要确保引入两个最核心的依赖：
			- Spring Cloud Gateway：提供网关的核心功能。
			- Nacos Discovery：让你的网关能连接到 Nacos 注册中心，并找到其它微服务。
		- ```xml
		  <dependencies>
		      <dependency>
		          <groupId>org.springframework.cloud</groupId>
		          <artifactId>spring-cloud-starter-gateway</artifactId>
		      </dependency>
		      <dependency>
		          <groupId>com.alibaba.cloud</groupId>
		          <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
		      </dependency>
		  </dependencies>
		  ```
	- **第三步：编写网关配置信息**
		- 在网关项目的 `resources` 目录下，修改 `application.yml` 文件。
		- ```yaml
		  # ===================================================================
		  #                      服务器配置 (Server Configuration)
		  # ===================================================================
		  # 服务器端口，我们的网关将运行在这个端口上
		  server:
		    port: 8080 # 网关对外暴露的端口
		  
		  # ===================================================================
		  #                      Spring 应用配置 (Application Configuration)
		  # ===================================================================
		  spring:
		    application:
		      name: gateway-service # 给网关服务自己起个名字
		  
		    # ===================================================================
		    #                      Spring Cloud 配置 (Cloud Configuration)
		    # ===================================================================
		    cloud:
		      # --- Nacos 服务发现配置 ---
		      nacos:
		        discovery:
		          server-addr: localhost:8848 # 你的 Nacos 服务器地址
		  
		      # ===================================================================
		      #                      网关核心配置 (Gateway Configuration)
		      # ===================================================================
		      gateway:
		        # --- 路由规则集合 (A Collection of Route Definitions) ---
		        routes:
		          # 路由规则 1: 用户服务
		          # -------------------------------------------------------------------
		          - id: user_route                                  # 路由的唯一标识，自定义，不要重复
		            uri: lb://userservice                           # 🎯 目标地址: 通过负载均衡(lb)转发到名为 "userservice" 的服务
		  
		            # --- 断言: 请求必须满足的条件 (Predicates) ---
		            predicates:
		              - Path=/user/** #   断言条件: 当请求路径匹配 /user/** 模式时，此路由生效
		  
		            # --- 过滤器: 请求在转发前后经过的处理 (Filters) ---
		            filters:
		              # 过滤器 1: 剥离路径前缀 (StripPrefix)
		              # 作用:   从请求路径中移除指定数量的前缀。设置为1，意味着会移除路径的第一个部分。
		              # 示例:   客户端请求 /user/profile/1 -> 网关处理后 -> 转发给 userservice 的路径将变为 /profile/1
		              # 场景:   这是最常用的过滤器之一，用于隐藏服务在网关层的路径前缀，让后端服务专注于自身业务路径。
		              - StripPrefix=1
		  
		              # 过滤器 2: 添加请求头 (AddRequestHeader)
		              # 作用:   向转发给后端服务的请求中，添加一个自定义的请求头。
		              # 格式:   AddRequestHeader=HeaderName, HeaderValue
		              # 示例:   所有经此路由的请求，都会被加上 "X-Request-Source: gateway" 的请求头。
		              # 场景:   用于服务间的来源识别、信息追踪、传递元数据等。
		              - AddRequestHeader=X-Request-Source, gateway
		  
		          # 路由规则 2: 订单服务 (示例)
		          # -------------------------------------------------------------------
		          # - id: order_route
		          #   uri: lb://orderservice
		          #   predicates:
		          #     - Path=/order/**
		          #   filters:
		          #     - StripPrefix=1
		  ```
- 过滤器
  heading:: true
	- **概念：**
		- 在请求真正被后端服务处理之前，或者处理完成之后，拦下来做一些额外的事情，比如检查权限、记录日志、修改请求内容，或者给响应加点信息。
		- 当一个请求进入网关时，会依次经过由多个过滤器组成的检查流水线。每个过滤器负责执行一项特定的操作：
			- 有的负责身份验证（认证过滤器）
			- 有的记录请求信息（日志过滤器）
			- 有的为请求添加标识（添加请求头的过滤器）
		- 只有通过所有检查的请求，才能被转发到后端的微服务进行处理。
		  id:: 6852c79f-8360-477e-81a5-81510ed72277
	- **两大类型的过滤器：**
		- **路由过滤器：**
			- **作用范围：**只对某一条特定的路由规则生效。
			- **配置方式：**在 `application.yml` 文件中，写在具体路由的 `filters` 节点下。
		- **全局过滤器：**
			- **作用范围：**对所有的路由规则都生效。
			- **配置方式：**通过编写 Java 代码实现 `GlobalFilter` 接口，并注册为一个 Spring Bean。
	- **示例：编写一个自定义的全局认证过滤器**
		- **目标：**检查所有请求的请求头中是否包含一个名为 `Authorization` 的 Header，并且其值必须为 `admin`。如果校验不通过，就拦截请求，返回 401 未授权；如果通过，则放行。
		- ```java
		  package com.example.gateway.filters;
		  
		  import org.springframework.cloud.gateway.filter.GatewayFilterChain;
		  import org.springframework.cloud.gateway.filter.GlobalFilter;
		  import org.springframework.core.Ordered;
		  import org.springframework.http.HttpStatus;
		  import org.springframework.http.server.reactive.ServerHttpRequest;
		  import org.springframework.http.server.reactive.ServerHttpResponse;
		  import org.springframework.stereotype.Component;
		  import org.springframework.web.server.ServerWebExchange;
		  import reactor.core.publisher.Mono;
		  
		  @Component // 关键：将这个过滤器注册为 Spring 的一个组件 (Bean)，不加这个注解，过滤器不会生效。
		  public class AuthGlobalFilter implements GlobalFilter, Ordered {
		  
		      /**
		       * 核心的过滤逻辑就在这里
		       * @param exchange 包含了请求和响应的上下文，你可以从它这里拿到 request 和 response。
		       * @param chain 过滤器链，用于将请求传递给下一个过滤器
		       * @return Mono<Void> 表示这是一个响应式的异步处理
		       */
		      @Override
		      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
		          // 1. 获取请求对象
		          ServerHttpRequest request = exchange.getRequest();
		          // 2. 获取响应对象
		          ServerHttpResponse response = exchange.getResponse();
		  
		          // 3. 从请求头中获取 'Authorization' 的值
		          String token = request.getHeaders().getFirst("Authorization");
		  
		          // 4. 编写我们的认证逻辑
		          //    为了演示，我们这里写一个简单的逻辑：token 必须是 "admin"
		          if (!"admin".equals(token)) {
		              // 5. 认证失败，拦截请求
		              System.out.println(">> 认证失败，请求被拦截！");
		              // 设置响应的状态码为 401 UNAUTHORIZED (未授权)
		              response.setStatusCode(HttpStatus.UNAUTHORIZED);
		              // 结束请求处理，直接返回响应
		              return response.setComplete();
		          }
		  
		          // 6. 认证成功，放行请求
		          //    调用 chain.filter(exchange) 将请求传递给下一个过滤器，如果这是最后一个过滤器，请求就会被发往后端的微服务。
		          System.out.println(">> 认证成功，请求被放行！");
		          return chain.filter(exchange);
		      }
		  
		      /**
		       * 指定过滤器的执行顺序
		       * @return 返回一个整数，值越小，优先级越高
		       */
		      @Override
		      public int getOrder() {
		          return -1; // 我们希望这个认证过滤器在所有过滤器中拥有较高的优先级，所以给一个负数值
		      }
		  }
		  ```
-