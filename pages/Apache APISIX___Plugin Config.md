## 是什么？有什么用？
	- Plugin Config 本质上是一组插件配置的预设模板，是可在多个路由间复用的插件组。
	- 你可以在 `plugin_configs/{id}` 里写一整坨 `plugins`，然后在多个 Route/Service 上通过 `plugin_config_id` 复用。
- ## Plugin Config API
	- **获取全部插件配置：**`GET`，`/apisix/admin/plugin_configs`，无请求体
	- **创建插件配置：**`PUT`，`/apisix/admin/plugin_configs/{id}`，有请求体
	- **删除插件配置：**`DELETE`，`/apisix/admin/plugin_configs/{id}`，无请求体
	- **更新插件配置：**`PATCH`，`/apisix/admin/plugin_configs/{id}`，有请求体
	  collapsed:: true
		- 更新指定现有插件配置的选定属性。若要删除某一属性，将该属性的值设为 `null` 即可。
	- **请求体：**
		- `plugins`：要执行的插件们。
- ## 注意
	- 一个 Route 只能有一个 `plugin_config_id`，不能在同一个 Route 上挂两个不同的 `plugin_config`。
	  logseq.order-list-type:: number
	- `plugin_config_id` 和 `plugins` 属性平级。
	  logseq.order-list-type:: number
	- **插件配置的合并：**
	  logseq.order-list-type:: number
		- 先把 `plugin_config` 里的 `plugins` 当作“默认值”挂上去。
		- 再来看 Route 自身的 plugins 配置规则：
			- 如果 Route 中已存在同名插件，则 Route 的配置会覆盖全局 `plugin_config` 中对应的同名插件配置；
			- 如果 Route 中不存在该插件，即使用 plugin_config 中的配置。
		- 冲突时 Route `plugins` 优先。
	- **关闭导入的插件：**导入 `plugin_config` 之后，在某条 Route 的 `plugins` 里写同名插件，配上 `"disable": true`，就能只在这一条 Route 上把这个插件关掉。
-