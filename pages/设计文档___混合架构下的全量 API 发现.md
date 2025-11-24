## 目标
	- 为了让 RBAC 能运作，必须拿到全量列表。
- ## 问题
	- 系统接口定义来源不统一，分成了两种：
	  logseq.order-list-type:: number
		- Proto：定义在 `.proto` 文件中，容易提取。
		  logseq.order-list-type:: number
		- 原生 Handler：直接写在 Go 代码里，不走 Kratos 的生成逻辑。
		  logseq.order-list-type:: number
	- 系统在不断演进，接口会改变。
	  logseq.order-list-type:: number
		- “只增不减”会让数据库变成垃圾场。
		  logseq.order-list-type:: number
		- 接口路径变更如果不处理，会导致原本配置好的权限突然失效。
		  logseq.order-list-type:: number
- ## 解决方案 A
	- 在服务启动之前，读取 `Server` 对象的路由列表，并发送到 RBAC 数据库（`url_resource` 服务）。
	  logseq.order-list-type:: number
		- 为了能处理 HTTP 请求，底层的 HTTP Server 必须知道完整的路由信息。无论这些 API 路径是通过 Proto 还是 Native 的方式注册的，最终都会被注册到同一个 `router` 中。
		- Kratos 的 HTTP Server 有 `WalkRoute` 方法，它会遍历路由树的每个路由。
	- 每次服务启动时，把数据库中存储的接口列表和代码中当前的接口列表进行比对，覆盖数据库的旧状态。
	  logseq.order-list-type:: number
		- **代码里有，数据库没：**INSERT。为了避免重复，数据库的 `(method, path)` 应该是唯一索引。
		- **数据库有，代码里没：**当检测到接口消失时，将数据库中的接口置为废弃（`Status = 3`），INSERT 新接口。
			- **前端 RBAC 界面表现：**可以在权限配置页面把这些废弃接口置灰，并提示“该接口已废弃”。管理员看到后，手动把该接口的权限迁移到新接口，然后再手动清理。
		- **两边都有：**UPDATE 数据库中的接口。
	- 多实例部署的“并发竞争”问题：多实例都去运行 `SyncAPIs`，导致数据库不必要的压力或重复插入报错。
	  logseq.order-list-type:: number
		- 利用 Redis 的分布式锁：只有抢到锁的那个实例才有资格执行 Sync 逻辑，其它的实例跳过。
-