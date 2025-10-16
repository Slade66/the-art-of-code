- **是什么？有什么用？**
	- vmagent 是一个轻量级、高性能、功能强大、独立的代理程序，专门负责收集、处理和转发监控指标。
	- `vmagent` 是一个专注于“指标采集和转发”的指标采集程序，它将 Prometheus 的采集功能独立出来并做了大量优化和功能增强，使其更高效、更可靠、更灵活。
	- 可以从各种数据源（如 node_exporter）拉取 (pull) 指标，或者接收推送 (push) 来的指标。作为一个中心化的数据收集点，接收来自不同协议的数据，统一处理后，再转发给后端的存储系统。
	- 在收集到指标后，它可以进行**过滤 (filtering)、重标签 (relabeling) 和聚合 (aggregation)**，然后将处理后的数据高效地发送到一个或多个远程存储系统，比如 VictoriaMetrics 或任何支持 Prometheus `remote_write` 协议的系统。
- 主要功能：
	- **数据处理：**
		- **重标签 (Relabeling)**：可以像 Prometheus 一样，在数据发送前添加、删除或修改指标的标签。
		- **过滤 (Filtering)**：可以根据标签丢弃不需要的指标，只发送有用的数据。
		- **流式聚合 (Stream Aggregation)**：可以在数据发送前进行预聚合，比如计算 `rate`、`sum` 等，从而减少存储压力和查询时的计算量。
	- **磁盘缓冲**：当远程存储不可用时，`vmagent` 会将收集到的指标**缓冲到本地磁盘**，并在连接恢复后自动重新发送，确保数据不丢失。
	- 数据路由：
		- **复制 (Replication)**：可以将同一份数据同时发送到**多个**远程存储系统，实现数据备份和高可用。
		- **分片 (Sharding)**：可以将不同的指标数据**分散地**发送到多个远程存储系统，实现水平扩展和负载均衡。
		- **按规则拆分 (Splitting)**：可以根据标签规则，将符合不同条件的指标发送到不同的目的地。例如，`env=prod` 的指标发到生产环境集群，`env=dev` 的发到开发环境集群。
- 应用场景：
	- **作为 Prometheus 的轻量级替代品**：如果你只使用 Prometheus 来抓取指标并将其发送到远程存储，那么 `vmagent` 是一个更高效、资源占用更少的完美替代方案。
	- **大规模指标抓取**：当需要抓取的目标 (targets) 数量非常庞大时，可以部署多个 `vmagent` 实例组成集群，分担抓取任务。
- HTTP 服务发现
	- 它允许 `vmagent` 通过访问一个 HTTP URL 来动态获取要抓取 (scrape) 的目标列表。
	- 你只需要维护一个能返回特定格式 JSON 的 HTTP 端点，就可以集中式、动态地管理所有监控目标，而无需修改和重启 `vmagent`。
	- 动态管理抓取目标特别适合服务实例经常变化的环境。这样你就不需要每次增删服务时都去修改 `vmagent` 的静态配置文件并重启它了。
	- 工作流程：
		- **定期轮询 (Polling)**：`vmagent` 会根据启动时设置的 `-promscrape.httpSDCheckInterval` 参数（或 Prometheus 的 `scrape_config` 中的 `refresh_interval`），周期性地向你在配置文件中指定的 `url` 发送一个 HTTP GET 请求。
		- **获取目标列表**：该 `url` 对应的服务端点必须返回一个特定结构的 **JSON 数组**。如果请求失败，`vmagent` 会在 `promscrape_discovery_http_errors_total` 指标中记录错误。
		- **解析 JSON**：`vmagent` 解析返回的 JSON 数据。JSON 中的每个对象都定义了一组具有共同标签的目标。
			- `targets` 数组中的每个字符串（通常是 `"<host>:<port>"` 格式）会被解析为一个**临时的 `__address__` 标签**，这就是抓取目标的地址。
			- `labels` 对象中的每一对键值，会被解析为以 `__meta_` 为前缀的**元数据标签**。例如，`"env": "production"` 会变成一个名为 `__meta_env` 的临时标签。
	- **JSON 响应格式要求：**
		- ```json
		  [
		    {
		      "targets": [ "<host1>:<port1>", "<host2>:<port2>" ],
		      "labels": {
		        "<labelname1>": "<labelvalue1>",
		        "<labelname2>": "<labelvalue2>"
		      }
		    },
		    {
		      "targets": [ "<host3>:<port3>" ],
		      "labels": {
		        "<labelname1>": "<labelvalue3>"
		      }
		    }
		  ]
		  ```
		- **`targets`：**一个字符串数组，要去哪里抓取指标。
		- **`labels`：**一个对象，包含了要应用到这组 `targets` 上的所有标签。
	- 实现步骤：
		- **创建一个 HTTP 端点 (Endpoint)**：这个端点返回一个特定格式的 JSON 数组，其中包含了所有需要抓取的目标 (targets) 列表。
		- **配置 `vmagent`**：让 `vmagent` 去定期访问这个 HTTP 端点，动态获取最新的目标列表。
			- ```yaml
			  scrape_configs:
			    - job_name: 'services-discovered-via-http'
			  
			      # 指定使用 HTTP 服务发现
			      http_sd_configs:
			        - url: 'http://192.168.1.100:8000/vmagent_targets.json' # <-- 替换成你 HTTP 服务器的实际 IP 和端口
			  ```
		- 运行 `vmagent`
-