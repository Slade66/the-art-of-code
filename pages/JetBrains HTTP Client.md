- **应用场景：**
	- **测试你的服务**：
		- 在完成一个功能后，你可能需要验证它是否能正常工作。例如，如果功能是返回用户数据，你可以使用这个工具发起请求，查看是否返回了正确的数据。
	- **测试第三方的服务**：
		- 有时候你的服务需要调用外部接口（如天气预报API）。在编写代码之前，你可以先使用这个工具了解外部服务的使用方式，明确接口所需的参数和返回的数据格式，这样可以帮助你更准确地编写代码。
		- 当你的程序调用外部接口失败时，可以通过 HTTP Client 独立向该接口发送请求。如果请求成功，说明可能是你的代码有问题；如果请求也失败，那就可能是外部接口或网络连接存在问题。
- **如何在 GoLand 中创建 HTTP 请求文件？**
	- **临时文件（Scratch Files）**
		- 这就像草稿纸，适合在开发过程中临时测试某个接口，无需将其保存到项目中，也不会成为项目代码的一部分。
		- 在日常开发中，你可以经常使用 Scratch Files 来快速测试服务接口。
	- **正式文件（Physical Files）**
		- 它们是项目的一部分，随着项目一起保存并提交到版本控制（如 Git），非常适合用来记录、测试和验证 API 接口，同时也可以作为团队内部的 API 文档。
		- 当 API 接口稳定，或需要为某个重要模块编写测试用例时，应考虑使用 Physical Files。这样可以将它们提交到 Git 仓库，方便团队其他成员查阅和复用，也是一种有效的文档化方式。
- **编写 HTTP 请求**
	- **语法：**
		- ```text
		  Method Request-URI HTTP-Version
		  Header-field: Header-value
		  
		  Request-Body
		  ```
	- **示例：**
		- ```http
		  GET https://api.example.com/users/1 HTTP/1.1
		  Host: api.example.com
		  Accept: application/json
		  
		  ### 这是一条 GET 请求示例
		  // 这也是一条注释
		  ```
	- **详解：**
		- **`###`**：用于分隔多个请求，每个请求之间用 `###` 隔开。该符号本身无实际意义，仅帮助 GoLand 理解文件中包含多个请求，并确保它们正确分隔。
		- **注释**：可以在 `###` 后面或请求的任何一行前面使用 `#` 或 `//` 添加注释，以便自己或他人理解请求的作用。
		- **请求行**：
			- **`Method`**：HTTP 方法，如 `GET`、`POST`、`PUT`、`DELETE` 等，需使用大写字母。
			- **`Request-URI`**：请求路径，例如 `/users/1` 或 `/products`。
			- **`HTTP-Version`**：HTTP 协议版本，通常为 `HTTP/1.1`。
		- **请求头（`Header-field: Header-value`）**：
			- `Content-Type`：指定发送数据的格式，如 `application/json` 或 `application/x-www-form-urlencoded`。
			- `Authorization`：如果 Kratos 微服务需要认证（例如 JWT Token），可在此处添加认证信息，如 `Bearer your_jwt_token`。
			- 其他自定义请求头...
		- **空行**：请求头与请求体之间必须有一个空行。
		- **请求体（`Request-Body`）**：对于 `POST` 或 `PUT` 请求，需要在此处写入发送的数据，如 JSON 格式的内容。
- **命名请求**：
	- 为了在运行/调试配置、`Search Everywhere` 和 `Run Anything` 中更方便地找到请求，可以为请求命名。
	- **命名请求有几种方式：**
		- 在请求上方使用 `###` 加上名称。
		- 使用 `# @name` 加上名称。
		- 使用 `# @name =` 加上名称。
		- **示例：**
			- ```text
			  ### 获取用户列表
			  GET https://api.example.com/users
			  ```
			- `### 获取用户列表` 即为该请求的名称。
	- **名称处理规则：**
		- 如果一个请求没有名称，GoLand 会使用它在请求文件中的位置作为名称（例如 `#1`）。
		- 如果一个请求文件中包含多个同名的请求，GoLand 会在每个同名请求的名称后面附加其位置编号，使其唯一，方便您在 `Services` 工具窗口、运行/调试配置等地方找到。
- **快速创建 HTTP 请求**：
	- **通过菜单栏**：
		- 在 GoLand 菜单栏中点击 `Tools` -> `HTTP Client` -> `Create Request in HTTP Client`。
		- 如果当前已经打开 `.http` 文件，它会直接在该文件中插入一个请求模板。
		- 如果未打开文件，它会新建一个临时 `.http` 草稿文件（Scratch file）。
	- **编辑器面板上的 `+` 号**：
		- 在 GoLand 编辑器中，点击请求面板左上角的 `+` 图标。
		- 这会弹出一个菜单，让你选择要添加的请求类型（例如 `GET Request`、各种类型的 `POST Request` 等）。
	- **万能的 Live Templates（动态模板）**：
		- 这类似于一个代码片段库。
		- 在编辑器中，按 `Ctrl + J`（macOS 上为 `Cmd + J`），即可弹出可用模板列表。
		- **举例：**
			- 输入 `gtr` 然后按 `Tab` 键，立即展开为一个简单的 `GET` 请求模板。
			- 输入 `mptr` 然后按 `Tab` 键，它会展开为一个处理文件上传的 `multipart/form-data` 的 `POST` 请求模板。
- **处理 `application/x-www-form-urlencoded` 内容类型：**
	- 使用 `application/x-www-form-urlencoded` 内容类型时，需特别注意请求体中的一些特殊字符处理。此内容类型通常用于提交 HTML 表单数据。
	- 其数据格式类似于 URL 查询字符串，即 **`key1=value1&key2=value2`**，其中 **`=`** 用于分隔键和值，**`&`** 用于分隔不同的键值对。
	- 如果键或值中包含 `%`、`&`、`=` 或 `+` 等字符，可能会引发歧义。例如，"开发+测试" 中的 `+` 会与空格冲突，"A&B" 中的 `&` 会被误认为是分隔符。为避免此问题，GoLand 的 HTTP Client 要求对这些特殊字符进行转义，方法是在字符前加 `%`。例如，`value%+value` 会转义为 `value%2Bvalue`，`value%&value` 会转义为 `value%26value`。
	- **示例：**
		- 如果希望服务器接收到以下数据：`field1=value+value&field2=value&value`
		- 在 GoLand 的 HTTP Client 中，请求体应写为：
		- ```text
		  POST https://ijhttp-examples.jetbrains.com/post
		  Content-Type: application/x-www-form-urlencoded
		  
		  field1=value%+value&field2=value%&value
		  ```
- **GET 请求的简写形式：**
	- 对于 `GET` 请求，您可以省略请求方法 `GET`，只指定 URI。GoLand 会默认将其识别为 `GET` 请求。
	- **示例：**
		- ```http
		  # 这是一个简写的 GET 请求，获取首页内容
		  https://example.com/
		  
		  ###
		  # 这是另一个简写的 GET 请求，获取用户个人资料
		  https://api.yourdomain.com/profile
		  ```
- **将长请求分解为多行：**
	- **路径分行：**
		- ```http
		  GET http://example.com:8080
		      /api
		      /html
		      /get
		      ?id=123
		      &value=content
		  ```
	- **查询参数分行：**
		- ```http
		  GET https://example.com:8080/api/get/html?
		      firstname=John&
		      lastname=Doe&
		      planet=Tatooine&
		      town=Freetown
		  ```
	- **`application/x-www-form-urlencoded` 请求体分行：**
		- ```http
		  POST https://ijhttp-examples.jetbrains.com/post
		  Content-Type: application/x-www-form-urlencoded
		  
		  key1 = value1 &
		  key2 = value2 &
		  key3 = value3 &
		  key4 = value4 &
		  key5 = value5
		  ```
- **提供请求消息体：**
	- **直接在请求中编写请求体：**
		- ```http
		  // 请求体直接在请求中提供
		  POST https://example.com:8080/api/createItem HTTP/1.1
		  Content-Type: application/json
		  Cookie: session_id=abc123xyz
		  
		  {
		    "itemName": "新商品",
		    "price": 99.99,
		    "description": "这是商品的描述"
		  }
		  ```
	- **从文件中读取请求体：**
		- 要从文件中读取请求体，请键入 `<` 符号，后跟文件路径。这对于发送大型请求体或需要重用请求体内容时非常有用。
		- **示例：**
			- 假设您有一个 `input.json` 文件，内容如下：
				- ```json
				  // input.json
				  {
				    "reportTitle": "月度销售报告",
				    "data": [
				      {"product": "A", "sales": 100},
				      {"product": "B", "sales": 150}
				    ]
				  }
				  ```
			- 在 HTTP 请求文件中：
				- ```http
				  // 请求体从文件中读取
				  POST https://example.com:8080/api/reports
				  Content-Type: application/json
				  
				  < ./input.json
				  ```
- **使用 `multipart/form-data` 内容类型：**
	- `multipart/form-data` 是一种 HTTP `Content-Type`，主要用于在 HTTP 请求中发送包含多种类型数据的表单。最常见的应用场景是文件上传，但它也可以同时包含普通的文本字段。
	- 它的核心思想是将整个请求体分解成多个独立的“部分”（part），每个部分都有自己的内容类型和名称，并且部分之间通过一个唯一的边界字符串（boundary）分隔。
	- **为什么要用它？**
		- 传统的 `application/x-www-form-urlencoded` 适合发送简单的键值对文本数据，但无法有效地处理二进制文件。`multipart/form-data` 则解决了这个问题，允许你在一个请求中同时发送文本字段和文件。
	- **怎么用？**
		- **设置 `Content-Type` 头：**
		  logseq.order-list-type:: number
			- 在 HTTP 请求头中，必须将 `Content-Type` 设置为 `multipart/form-data`，并紧接着指定一个 `boundary` 字符串。`boundary` 是用来分隔各个数据部分的标识符，必须是一个在整个请求体中不会重复的唯一字符串，否则会导致解析错误。
			- **示例：**
				- ```http
				  POST https://example.com/api/upload HTTP/1.1
				  Content-Type: multipart/form-data; boundary=myCustomBoundaryString
				  ```
				- 这里的 `myCustomBoundaryString` 是我们自定义的边界字符串。通常为了确保唯一性，`boundary` 会是一个随机生成的长字符串。
		- **定义各个数据部分：**
		  logseq.order-list-type:: number
			- 每个数据部分都以 `--` 和 `boundary` 字符串开始，并在其内部包含该部分的具体数据。
			- **`--boundary`**：表示每个数据部分的开始。
			- **`Content-Disposition` 头：** 每个部分必需包含该头，它描述该部分是“表单数据”（`form-data`），并指定该字段的 `name`。
				- `name="fieldName"`：对应后端接收的表单字段名称。
				- `filename="fileName.ext"`（可选，通常用于文件）：如果该部分是文件，需包含 `filename` 参数，指明文件的原始名称。
			- **`Content-Type` 头（可选）：** 如果该部分是文件，通常会指定文件的 MIME 类型（如 `image/jpeg`、`application/pdf`、`text/plain` 等）。如果未指定，后端可能会尝试猜测或使用默认值。
			- **空行：** 所有头部信息之后必须有一个空行。
			- **数据内容：** 空行之后为该部分的实际数据。
		- **结束整个请求体：**
		  logseq.order-list-type:: number
			- `multipart/form-data` 请求的结束标志是 `--` 和 `boundary` 字符串，后面再加 `--`。
	- **示例：**
		- ```http
		  POST https://example.com/api/upload HTTP/1.1
		  Content-Type: multipart/form-data; boundary=boundary
		  
		  --boundary
		  Content-Disposition: form-data; name="first"; filename="input.txt"
		  
		  // The 'input.txt' file will be uploaded
		  < ./input.txt
		  
		  --boundary
		  Content-Disposition: form-data; name="second"; filename="input-second.txt"
		  
		  // A temporary 'input-second.txt' file with the 'Text' content will be created and uploaded
		  Text
		  --boundary
		  Content-Disposition: form-data; name="third";
		  
		  // The 'input.txt' file contents will be sent as plain text.
		  < ./input.txt --boundary--
		  ```
- **自定义 HTTP 请求超时**：
	- GoLand HTTP Client 默认有两种主要的超时设置：
		- **连接超时 (Connection Timeout):** 默认为 **60 秒**，即 GoLand 尝试与目标服务器建立 TCP 连接的最大时间。如果在该时间内未能建立连接，请求将会失败。
		- **读取超时 / 数据包超时 (Read Timeout / New Packets Timeout):** 默认为 **60 秒**，指连接建立后，GoLand 等待服务器发送下一个数据包（例如响应头或响应体一部分）的最长时间。如果在该时间内未收到任何数据，请求会失败。该超时设置会不断计时，每当接收到新数据时，计时器会重置。
	- **GoLand 提供了两种层级的超时配置方式：**
		- **请求级别自定义超时**：
			- 您可以在单个 HTTP 请求文件中使用注释标签设置超时。这种方式灵活，只对当前请求生效，不会影响其他请求。
			- **设置读取超时（`@timeout`）：**
				- 用于控制连接建立后等待新数据包的超时时间。
				- **语法：** `# @timeout <value> [unit]` 或 `// @timeout <value> [unit]`
				- **示例：**
					- ```http
					  # @timeout 600
					  GET example.com/api/long-response
					  ```
					- 这个例子将 `GET example.com/api/long-response` 的读取超时设置为 600 秒。
			- **设置连接超时（`@connection-timeout`）：**
				- 用于控制与服务器建立连接的超时时间。
				- **语法：** `# @connection-timeout <value> [unit]` 或 `// @connection-timeout <value> [unit]`
				- **示例：**
					- ```http
					  // @connection-timeout 2 m
					  GET example.com/api/unstable-connection
					  ```
					- 这个例子将 `GET example.com/api/unstable-connection` 的连接超时设置为 2 分钟。
			- **单位说明：**
				- 默认情况下，超时值以秒为单位。但您可以在数值后面明确指定单位：
					- `ms`：毫秒
					- `s`：秒 (默认)
					- `m`：分钟
		- **IDE 级别自定义超时**：
			- 若希望所有 HTTP 请求（包括 GoLand 与外部服务交互时发出的请求）都遵循特定的超时设置，可以在 IDE 级别进行配置。这通过修改 GoLand 的 VM 选项来实现。
			- **如何修改 VM 选项：**
				- 点击 `Help (帮助)` -> `Edit Custom VM Options (编辑自定义 VM 选项)`。
				- 这将打开 `.vmoptions` 文件，您可以在其中添加或修改以下参数：
				- ```text
				  # 配置连接超时
				  -Didea.connection.timeout=30000
				  # 配置读取超时
				  -Didea.read.timeout=120000
				  ```
			- **默认单位为毫秒：** 通过 VM 选项设置的超时值，默认单位是毫秒。
			- **请求级别覆盖 IDE 级别设置：** 如果在单个 HTTP 请求中使用了 `@timeout` 或 `@connection-timeout` 标签，它将覆盖 VM 选项中设置的相应超时。这意味着请求级别的设置优先级更高。
			- **全局影响：** VM 选项会影响 GoLand 整个应用程序的行为。这些超时不仅适用于 HTTP Client 中的请求，还会影响 GoLand 与网络交互时发出的所有 HTTP 请求（如检查更新、下载依赖、访问插件市场等）。因此，修改这些值时请谨慎操作。
-