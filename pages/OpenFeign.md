有什么用？
heading:: true
	- OpenFeign 用于服务间通信，自动集成了服务发现（从 Nacos 获取服务地址）和负载均衡（在多个实例中自动选择），使你可以像调用本地方法一样调用远程 HTTP 服务，极大地简化了代码。
- 怎么用？
  heading:: true
	- 在项目中引入依赖：`spring-cloud-starter-openfeign`。
	- 定义一个 Java 接口，并使用 `@FeignClient("service-name")` 注解指定要调用的服务名称，接口方法的签名需与目标服务的 Controller 保持一致。
	- 调用该接口方法时，OpenFeign 会自动完成服务发现、负载均衡、HTTP 请求构造和响应解析，让远程调用像本地方法一样简单。
-