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
	- **示例：**
		- ```text
		  ### 获取用户列表
		  GET https://api.example.com/users
		  ```
		- `### 获取用户列表` 即为该请求的名称。
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
-