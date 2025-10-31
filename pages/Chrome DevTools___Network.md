- **Record（录制）：**按下时是红色（🔴），表示“正在录制”网络请求。**确保它一直是红色的。**
- **Clear（清除）：**一键清空当前列表。在分析新操作前，先点它清屏。
- **Filter（过滤）：**
  collapsed:: true
	- `Fetch/XHR`
		- `Fetch/XHR` 是浏览器用来“异步请求 API”的通道。
		- 这会帮你过滤掉所有无关的资源，比如图片 (Img)、样式 (CSS)、脚本 (JS)。你的屏幕会瞬间清爽，只剩下 API 请求！
- **Preserve log（保留日志）：**
  collapsed:: true
	- 当你点击“登录”时，页面会跳转（刷新）。如果不勾选，F12 会自动清空上一个页面的所有请求记录。勾选后，你才能抓到那个关键的“登录” API 请求，否则它一闪就没了。
- **`Headers`：请求和响应的头。**
  collapsed:: true
	- `General`：常规。
		- `Request URL`：这是请求 API 的完整地址。
		- `Request Method`：这是请求 API 的方法。
		- `Status Code`：这是请求的响应码。
		- `Remote Address`：这是请求的实际地址。
	- `Request Headers`：请求头。这里是你（浏览器）发给服务器的信息。
		- 在这里找 `Cookie` 或 `Authorization`，看你登录后，后续请求是如何携带“身份凭证”的。
	- `Response Headers`：响应头。服务器返回给你的信息。
		- 登录响应里，这里可能会有 `Set-Cookie` 指令，这就是服务器在给你发“会话通行证”。
- **`Preview` (预览) - `Response` 的“美化版”**
  collapsed:: true
	- 如果 `Response` 返回的是 JSON，`Preview` 会帮你格式化好，让你看得更清楚。和 `Response` 内容一样。
- **`Payload` (载荷) - 看你“提交”了什么**
  collapsed:: true
	- 这个标签页会显示你发送给服务器的数据。
- **`Response` (响应) - 看服务器“返回”了什么**
  collapsed:: true
	- 服务器在处理完你的请求后，返回给你的数据。
-