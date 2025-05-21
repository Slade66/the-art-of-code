有什么用？
heading:: true
	- Spring Data Redis 是操作 Redis 的高层客户端，提供统一的 `RedisTemplate` 接口，简化了与 Redis 的交互。它屏蔽了底层客户端（如 Jedis、Lettuce）之间的 API 差异，使开发者无需关心具体实现，只需专注于业务逻辑。同时，底层实现可通过配置灵活切换，便于维护与扩展。
- 怎么用？
  heading:: true
	- 引入依赖
	  logseq.order-list-type:: number
		- ```maven
		  <dependency>
		      <groupId>org.springframework.boot</groupId>
		      <artifactId>spring-boot-starter-data-redis</artifactId>
		  </dependency>
		  ```
	- 配置 `application.yml`
	  logseq.order-list-type:: number
		- ```yml
		  spring:
		    data:
		      redis:
		        host: localhost
		        port: 6379
		        password: xxx
		        database: 0
		        timeout: 5000ms
		  ```
	- 注入 `RedisTemplate` 或 `StringRedisTemplate` 使用
	  logseq.order-list-type:: number
		- ((682d8809-e8f5-47e0-9255-6f8a89679e76))
		- ((682d8852-5a09-4e4c-82d4-24fe6546ef8f))
- `RedisTemplate`
  heading:: true
  id:: 682d8809-e8f5-47e0-9255-6f8a89679e76
	- `RedisTemplate` 是 Spring Data Redis 提供的一个通用 Redis 操作类，它封装了 Redis 的各种数据结构操作，用于在 Spring 应用中读写 Redis 数据。
	- 默认情况下，它采用 Java 原生的序列化机制，这会带来以下问题：
		- 数据以二进制形式存储在 Redis 中，不可读，调试困难；  
		  logseq.order-list-type:: number
		- 仅能在 Java 环境中反序列化，缺乏跨语言兼容性；  
		  logseq.order-list-type:: number
		- 序列化后的数据体积较大，占用更多存储空间。  
		  logseq.order-list-type:: number
	- 因此，在实际开发中，通常会通过自定义 Bean 的方式，修改 `RedisTemplate` 的序列化策略以提升可读性和兼容性。
- `StringRedisTemplate`
  heading:: true
  id:: 682d8852-5a09-4e4c-82d4-24fe6546ef8f
	- `StringRedisTemplate` 是 `RedisTemplate` 的简化版本，专用于处理字符串类型的 key 和 value。
	- 预设了字符串序列化器，写入 Redis 的内容为纯文本格式，便于调试与可视化。
	- 尽管 `StringRedisTemplate` 使用起来很方便，但它仅支持存储字符串类型的 key 和 value。如果需要保存 Java 对象，则必须手动进行对象与字符串之间的序列化和反序列化操作。
-