-
- Prometheus 负责**采集和存储**监控数据。它使用一种被称为 **Pull 模式** 的方式，主动从你的应用（称为 **Targets**）那里拉取数据。
-
- 一个完整的监控流程通常是这样的：**数据采集器 (Exporter) -> 数据存储和查询 (Prometheus) -> 数据可视化 (Grafana)。**
-
- Prometheus 定义了行业标准和核心概念，是监控系统的1111
- 监控系统核心概念
	- **指标 (Metrics)**: 理解什么是指标。它是一种在特定时间点采集的可度量数据，比如当前 CPU 使用率是 15%，内存剩余 4.2 GB。
		- 代表应用程序状态的数值数据，比如 `http_requests_total`（HTTP 请求总数）或 `go_goroutines`（Go 协程数量）。
	- **时间序列数据 (Time-Series Data)**: 你要处理的所有监控数据都是时间序列数据。它是由 `(指标名称 + 一组标签, 时间戳, 值)` 组成的数据流。
		- **指标名称 (Metric Name)**: 如 `node_cpu_seconds_total` (CPU 总使用时间)。
		- **标签 (Labels)**: 用来区分和筛选数据的键值对。这是监控系统的精髓！例如，`{instance="server1:9100", job="node_exporter"}` 就能让你精确地定位到 `server1` 这台机器上的 `node_exporter` 服务采集的数据。在你的项目中，`instance` 或自定义的 `host_id` 标签将是区分不同服务器的关键。
		- 年龄{人=“李雨泽”} 我要显示年龄数据，谁的年龄数据？label 缩小范围。
	- **数据模型 (Data Model)**: 了解 VictoriaMetrics (以及 Prometheus) 的数据模型。知道数据是如何被组织和存储的，可以帮你更有效地查询。
	- **Exporter (导出器)**: 这是监控数据的来源。它是一个运行在被监控主机上的程序，负责采集主机的各种状态（如 CPU、内存、磁盘、网络等），并将其转换为 VictoriaMetrics 能理解的格式，通过一个 HTTP 端点暴露出来。
		- **关键知识点**: 你必须了解 `node_exporter`。这是最常用
		  的用于采集主机物理指标（CPU、内存、磁盘等）的 Exporter。你需要知道它暴露了哪些关键指标，比如：
			- `node_cpu_seconds_total`: CPU 在各种模式下（user, system, idle...）花费的总秒数。这是一个 `Counter` 类型，只会增长。
			- `node_memory_MemTotal_bytes`: 总内存大小。
			- `node_memory_MemAvailable_bytes`: 可用内存大小。
			- `node_filesystem_avail_bytes`: 文件系统的可用空间。
			- `node_network_receive_bytes_total`: 网络接收的总字节数。
- PromQL
	- Prometheus 的查询语言，用于查询和处理时间序列数据。
	- **选择时间序列 (Selector)**: 如何通过指标名称和标签筛选数据。
		- `node_memory_MemAvailable_bytes{instance="server-A"}`
	- **范围查询 (Range Vector)**: 如何获取一个时间段内的数据，比如 `[5m]` 表示过去 5 分钟。
		- `node_cpu_seconds_total{mode="idle"}[5m]`
	- **聚合函数 (Aggregation)**: `sum()`, `avg()`, `count()`, `max()` 等。
		- `sum(node_cpu_seconds_total)`
	- **关键函数 (Key Functions)**:
		- `rate(v range-vector)`: 计算 `Counter` 类型指标在指定时间范围内的**平均每秒增长率**。这是计算 CPU 使用率、网络流量等指标的**标准方法**。
		- `irate(v range-vector)`: 计算瞬时增长率。对变化剧烈的指标更敏感。
		- `increase(v range-vector)`: 计算在指定时间范围内的总增长量。
- Exporter
	- 整个生态的数据来源入口。
- 怎么用？
	- ### 第一步：准备 Prometheus 配置文件 ( `prometheus.yml` )
		- ```yaml
		  global:
		    scrape_interval: 15s
		  
		  scrape_configs:
		    - job_name: 'prometheus'
		      # Prometheus 监控自己
		      static_configs:
		        - targets: ['localhost:9090']
		  
		    - job_name: 'node_exporter'
		      # 监控 WSL 主机性能指标
		      static_configs:
		        - targets: ['host.docker.internal:9100']
		  ```
		- 这里需要注意 **`host.docker.internal`** 这个特殊的域名。在 Docker Desktop 中，这个域名会自动解析为 Docker 宿主机的 IP 地址。这解决了 Prometheus 容器如何访问到 WSL 主机的问题。**Node Exporter** 默认监听在 9100 端口，所以我们这里配置了 `9100`。这个特殊的域名会解析为 Docker 宿主机的 IP 地址
	- ### 第二步：启动 Node Exporter 容器
		- Node Exporter 是一个专门用于收集系统指标（CPU、内存、磁盘、网络等）的工具。我们用 Docker 启动它。
		- ```bash
		  docker run -d \
		      --name node_exporter \
		      -p 9100:9100 \
		      --restart=always \
		      --net="host" \
		      --pid="host" \
		      quay.io/prometheus/node-exporter:latest \
		      --path.rootfs=/host
		  ```
	- ### 第三步：启动 Prometheus 容器
		- 进入到你存放 `prometheus.yml` 的目录，然后运行：
			- ```bash
			  docker run -d \
			      --name prometheus \
			      -p 9090:9090 \
			      -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
			      prom/prometheus
			  ```
		- http://localhost:9090
- 监控自己
	- Prometheus 自身会暴露大量的内部指标，
	- 你可以在浏览器中访问 `http://localhost:9090/metrics`，你会看到类似 `prometheus_build_info`、`prometheus_tsdb_head_chunks_total` 等以 `prometheus_` 开头的指标。这些就是 Prometheus 自身产生的、用于自我监控的数据。
-
- 服务发现机制
	- 让 Prometheus 自动找到要抓取（scrape）指标的目标（target）的机制，而无需手动在配置文件中列出每个目标的地址。
	- 当有新主机上线时，Prometheus 如何才能知道它的存在并开始抓取其指标。
	- Prometheus 的服务发现机制主要可以分为两大类：静态配置和动态服务发现。
	- 静态配置
		- 手动在 Prometheus 配置文件 `prometheus.yml` 的 `scrape_configs` 中，以静态列表的形式定义所有目标。
		- **缺点**：每次有新目标上线或下线，都需要手动修改配置文件并重启或重新加载 Prometheus。
		- ```yaml
		  scrape_configs:
		    - job_name: 'docker_hosts_static'
		      static_configs:
		        - targets: ['host-a:9100', 'host-b:9100']
		  ```
	- 文件服务发现
		- Prometheus 会监控一个或多个文件，当文件内容发生变化时，Prometheus 会自动更新其抓取目标列表。
		- **工作方式**：您需要定义一个文件路径，Prometheus 会定期检查该文件。您的外部程序（例如您的 Go 后端服务）可以负责生成和更新这个文件。
		- 当您新增主机时，您的后端服务可以向文件中写入新主机的地址，Prometheus 就会自动发现它。
		- `prometheus.yml` 配置：
			- ```yaml
			  scrape_configs:
			    - job_name: 'docker_hosts_file'
			      file_sd_configs:
			        - names: ['/etc/prometheus/targets/hosts.json']
			  ```
		- `hosts.json` 文件内容（由您的程序生成）：
			- ```json
			  [
			    {
			      "targets": ["host-a:9100"],
			      "labels": { "instance_name": "host-a" }
			    },
			    {
			      "targets": ["host-b:9100"],
			      "labels": { "instance_name": "host-b" }
			    }
			  ]
			  ```
	- HTTP 服务发现
		- HTTP 服务发现的本质是让 Prometheus 定期向一个指定的 HTTP API 端点发送请求，您的后端服务则负责响应这个请求，并返回一个 JSON 格式的、包含所有监控目标的列表。
		- Prometheus 会以 GET 方式请求您提供的 URL。
		- 这个过程就像是 Prometheus 在问您的系统：“喂，我现在需要监控哪些主机？”，而您的后端服务则实时地给出答案。
		- #### 1. Prometheus 的配置
			- 在 Prometheus 的 `prometheus.yml` 文件中，您需要添加一个 `http_sd_configs` 配置块。您只需要指定一个 URL 和一个轮询间隔。
			- ```yaml
			  scrape_configs:
			    - job_name: 'custom_targets'
			      http_sd_configs:
			        - url: 'http://your-backend-service/api/v1/prometheus/targets' # 您后端服务的 API 地址
			          refresh_interval: 30s # Prometheus 每 30 秒轮询一次
			      # 这里也可以加上 relabel_configs 来处理返回的 targets
			  ```
		- #### 2. 您的 Go 后端服务实现
			- 您需要使用 Go 语言（例如基于 Kratos 框架）实现一个 HTTP 端点。当 Prometheus 向这个端点发送 GET 请求时，您的服务需要执行以下操作：
				- **查询数据源**：从您的数据库中查询所有已注册的主机列表。
				- **构建 JSON 响应**：将查询结果转换成 Prometheus 期望的 JSON 格式。
				- **返回响应**：将 JSON 数据返回给 Prometheus。
					- 必须返回 **HTTP 200 OK** 状态码。
					- `Content-Type` 头部必须是 `application/json`。
					- 响应体必须是 **UTF-8 编码**的 JSON 格式。
					- 即使没有目标，也必须返回一个 **空列表 `[]`**，而不是其他状态码。
			- 期望的 JSON 格式
				- Prometheus 期望的 JSON 响应是一个包含零个或多个对象的数组。每个对象都必须包含一个 `targets` 数组和一个可选的 `labels` 对象。
				- `targets`：一个字符串数组，每个字符串都是一个监控目标的地址（例如 `host:port`）。
				- `labels`：一个键值对对象，其中的键值对会被附加到这个 target 集合中的所有指标上。
				- ```yaml
				  [
				    {
				      "targets": ["host-a:9100"],
				      "labels": { "instance_name": "host-a", "environment": "production" }
				    },
				    {
				      "targets": ["host-b:9100", "host-c:9100"],
				      "labels": { "instance_name": "host-b-c", "environment": "development" }
				    }
				  ]
				  ```
			- **Prometheus **不支持**增量更新。您的 API 每次都必须返回**完整的**目标列表，而不是只返回新增或变更的目标。**
			- **错误处理**：如果您的端点返回错误或不可用，Prometheus 会继续使用**上一次成功获取的目标列表**，直到下次成功获取。
-
- Prometheus 指标格式
	- 这个格式主要由三部分组成：**注释行** (`# HELP` 和 `# TYPE`) 和 **指标数据行**。
	- `# HELP` 注释：**解释这个指标是干什么用的**。
	- `# TYPE` 注释 指标的数据类型？Prometheus 主要有四种类型，`gauge`的值可以任意上升或下降，就像汽车的速度表或温度计一样。它表示一个**瞬时值**。
	- 指标数据行
		- ```text
		  npu_chip_info_health_status {container_name="",id="0",...}   1   1756891340680
		  └─────────┬─────────┘   └───────────────┬──────────────┘ └──┬──┘ └──────┬──────┘
		  指标名称 (Metric Name)          标签 (Labels)         值 (Value)   时间戳 (Timestamp)
		  ```
		- 指标名称：测什么？
		- 标签：测谁？标签是一些键值对，用来描述指标的**维度**，让你能够区分和筛选同名但来自不同来源的指标。这台机器上有 8 块 NPU 卡。如果没有标签，你只能得到一个笼统的“健康状态”，但你不知道是哪块卡。有了标签 `id="0"` 到 `id="7"`，你就可以精确地知道**每一块特定 NPU 卡**的健康状态。
		- 值：这就是在当前这个时间点，该指标的**实际测量值**。
		- **时间戳 (Timestamp)**: `1756891340680` Unix 时间戳（毫秒级）。它告诉我们这个值是在哪个时间点被采集的。
-
- 结构是这样的：
	- `# HELP`: 这一行是对指标的简单描述。
	- `# TYPE`: 这一行定义了指标的类型，比如 `gauge` (仪表盘，表示一个可增可减的瞬时值) 或 `counter` (计数器，表示一个只增不减的累计值)。
	- `metric_name{label1="value1", label2="value2"}`: 这是指标本身，包含了指标名称和一组键值对形式的标签（labels），标签可以让你对指标进行筛选和聚合。
	- `value`: 指标在当前时刻的值。
-
- 在 PromQL 中，指标名称本身就是最基础的筛选器
- 把 Prometheus 想象成一个巨大的数据库，里面存储了无数条**时间序列 (Time Series)**。每一条时间序列都是由两部分唯一确定的：
	- **指标名称** (e.g., `npu_chip_info_utilization_gauge`)
	- 一组**标签 (Labels)** 的键值对 (e.g., `{id="0", host="...", ...}`)
- 因为指标名称是基础筛选器，它会匹配所有同名的时间序列，无论它们的标签是什么。
-
- 下面这两行数据代表了两条完全不同的时间序列，尽管它们的名字相同：
	- npu_chip_info_utilization_gauge{host="...", id="0", ...} 31
	  npu_chip_info_utilization_gauge{host="...", id="1", ...} 30
- 它们的名字都是 `npu_chip_info_utilization_gauge`，但因为一个的 `id` 是 "0"，另一个是 "1"，Prometheus 就把它们当作两条独立的数据流来存储。
- 当你只输入 `npu_chip_info_utilization_gauge` 时，你告诉 Prometheus 的规则是：
	- "请找出所有**指标名称**为 `npu_chip_info_utilization_gauge` 的时间序列，并返回它们当前的值。"
- 这就像在一个 SQL 数据库中执行这样一个查询：
	- ```sql
	  SELECT * FROM metrics WHERE metric_name = 'npu_chip_info_utilization_gauge';
	  ```
- 这条 SQL 语句会返回所有 `metric_name` 这一列等于指定值的行，PromQL 的逻辑与此非常相似。
- 你可以在指标名称后面加上花括号 `{}` 来增加更多的筛选条件。
- 从简单到复杂，从易到难。
-