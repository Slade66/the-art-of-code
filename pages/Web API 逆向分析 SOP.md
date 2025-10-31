- **原则：**
	- 将自己模拟为前端（浏览器），欺骗后端（服务器），让它以为你就是它“认识”的那个前端。
- **工具：**
	- **浏览器开发者工具（F12）：**用于抓取请求和响应进行分析，Network 标签页一定要精通！
	- **API 客户端（Apifox）：**用于记录、复现和测试 API。
- **阶段一：先注册**
  collapsed:: true
	- **目标：**获取一个目标网站的登录账号。
	- 注册过程略...
- **阶段二：抓鉴权**
  collapsed:: true
	- **目标：**通过登录 API 获取登录凭证，这是之后访问其它 API 的钥匙。
	- **步骤：**
		- 打开目标网站的登录页，打开 F12，切换到 `Network` 面板，在过滤器中，选择 `Fetch/XHR`。
		- 输入用户名、密码，点击“登录”。
		- 此时 F12 里会捕获到一个（或多个）API 请求，通常是 `POST` 方法，URL 可能是 `/login`、`/api/auth/login` 之类的。
		- 分析这个登录请求，查看它的 `Response`。服务器会“吐”出你的凭证。
			- **Token**：在 `Response Body` 的 JSON 里，会有一个很长的字符串。
			- **Cookie**：在 `Response Headers` (响应头) 里，会有一个 `Set-Cookie` 指令。
		- 在 Apifox 中配置“钥匙”：
			- **如果是 Token：**在 Apifox 的“环境管理”中，创建一个全局变量（例如 `token`）。然后，在需要鉴权的 API 的 `Headers` 中统一添加：`Authorization` : `Bearer {{token}}`。
			- **如果是 Cookie：**把 `Set-Cookie` 里的 `key=value` 键值对，添加到 Apifox 的 `Cookie` 管理中。
			- **CSRF 令牌：**如果是 Cookie 鉴权，经常会伴随 `X-CSRF-Token` 或 `X-Token` 这种 Header。这个 Token 通常也在登录响应中，或者在主页的 HTML `<meta>` 标签里。你必须把它也加到 Apifox 的 Header 中。
- **阶段三：抓接口**
  collapsed:: true
	- **目标：**
		- 搞清楚“做什么操作 = 调用哪个 API”，以及搞懂这个 API 的端点、参数和响应结构。
		- **产出：**把所有 API 固化为能调通、详细的 Apifox 文档。
	- **步骤：**
		- **核心思路：**在浏览器 F12 的 Network 页签下，每当你在页面上做一件事（加载、点击、切换、保存），就去观察新出现的 API 请求，然后立刻导入到 Apifox 中进行复现。
		- 打开 F12 -> Network 页签。
		  logseq.order-list-type:: number
		- 在过滤器 (Filter) 里，勾选 `Fetch/XHR`。这会帮你过滤掉所有图片、CSS、JS 等静态资源，只看 API 请求。
		  logseq.order-list-type:: number
		- 在页面上点击某个按钮。
		  logseq.order-list-type:: number
		- 用 F12 观察捕获到的请求，搞清楚它的端点、请求参数和响应结构。
		  logseq.order-list-type:: number
		- **导入到 Apifox：**
		  logseq.order-list-type:: number
		  collapsed:: true
			- **导入：**
				- 在 F12 的 Network 面板，右键点击一个你分析好的请求 -> `Copy` -> `Copy as cURL (bash)`。
				- 在 Apifox 中，点击“导入” -> “cURL”，粘贴即可。它会自动帮你填好所有 Headers、Body、Params。
				- 点“发送”看能不能拿到一样的 JSON 响应。
			- **编写文档：**
				- 为每个 API 编写“人类可读”的名称（如：“获取当前告警列表”）。
				- 为 Request Body 和 Response Body 的每个 JSON 字段添加中文注释。
			- **注意：不要硬编码！**
				- 把 URL 的公共前缀（如 `https://api.shuzhihui.com`）替换为 `{{baseUrl}}` 环境变量。
				- 把 Token 替换为 `{{token}}` 环境变量。
				- 把路径中的 ID（如 `/api/rules/uuid-12345`）替换为路径参数（如 `/api/rules/:ruleId`），并在 Apifox 中填入示例值。
		- **在 Apifox 里进行复现：**
		  logseq.order-list-type:: number
		  collapsed:: true
			- 修改参数的值，点击“发送”，看看接口的行为是否和预期一样，验证猜想。
				- **Create (增) - `POST`**
					- **你的操作：** 在页面上点击“新建”、“保存”、“提交”、“创建”。
					- **F12 观察：** 捕获一个 `POST` 请求。
					- **分析重点：** `Request Body (Payload)`。这里面的 JSON 结构，就是“创建”这个事物所需要的所有数据。
					- **复现关键：**修改 `Request Body` 里的 JSON 值，点击“发送”，看数据是否真的被创建了。
				- **Get (查) - `GET` (带 ID)**
					- **你的操作：** 点击列表中的某一项，进入“详情页”。
					- **F12 观察：** 捕获一个 `GET` 请求，URL 特征非常明显，如：`/api/v1/rules/uuid-12345` 或 `/api/v1/items?id=123`。
					- **分析重点：** `Path (路径)` 或 `Query (URL)` 参数中的 `id`。
					- **复现关键：**尝试把 `id` 换成另一个，看是否能拉取到另一条数据的详情。
				- **Update (改) - `PUT` / `PATCH`**
					- **你的操作：** 在详情页修改了某个表单，点击“更新”或“保存”。
					- **F12 观察：** 捕获一个 `PUT` 或 `PATCH` 请求，URL 同样会带着 `id`。
					- **分析重点：** `Request Body (Payload)`。这个 JSON 就是“更新”需要的数据。
					- **复现关键：**修改 `Request Body`，发送请求，看数据是否真的被修改了。
				- **Delete (删) - `DELETE`**
					- **你的操作：** 点击“删除”按钮。
					- **F12 观察：** 捕获一个 `DELETE` 请求，URL 带着 `id`。
					- **分析重点：** 通常没有 `Request Body`，只需要 `id`。
					- **复现关键：**把 `id` 换成你想删的数据的 ID，发送请求，看看数据有没有被删除。
				- **List (查 - 列表) - `GET`**
					- **你的操作：** 打开一个列表页面、翻页、搜索、筛选。
					- **F12 观察：** 捕获一个 `GET` 请求，如 `/api/v1/alarms`。
					- **分析重点：** `Query Parameters (URL 参数)`。这是“精华”所在。
						- 翻页：`?page=2&size=20`
						- 搜索：`?keyword=cpu_usage`
						- 筛选：`?status=active&level=critical`
						- 排序：`?sort_by=time&order=desc`
					- **复现关键：**测试这些 URL 参数的组合，看返回的 `Response Body` 是否符合预期。
				-
-