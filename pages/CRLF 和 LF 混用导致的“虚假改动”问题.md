- 使用 Git 管理文件时，你可能会遇到这样的情况：打开文件后未做任何修改，但 Git 却提示文件已被改动。这类“虚假改动”通常是由于不同操作系统对换行符（EOL）的处理方式不同所致。
- 当本地文件使用的换行符（如 CRLF）与 Git 仓库中存储的格式（如 LF）不一致时，Git 在执行 diff 操作时会将其识别为内容变化。某些编辑器或 Git 本身可能会在不经意间根据系统规范自动转换换行符，导致虽然文件内容未变，底层字节却发生变化，从而被 Git 误判为文件已被修改。
- 不同系统的换行符差异
  heading:: true
	- 不同操作系统对换行的定义存在差异：
		- Windows 使用 `CRLF`（回车 + 换行，`\r\n`）；
		- Unix/Linux/macOS 使用 `LF`（换行，`\n`）。
	- 因此，在跨平台开发或多人协作时，如果未统一换行符规范，容易导致换行符混用的问题。
- 解决方案：统一换行符策略
  heading:: true
	- 设置  `.gitattributes`  文件
	  heading:: true
		- 在项目根目录下新建或修改 `.gitattributes` 文件，添加以下内容：
		- ```.gitattributes
		  * text=auto eol=lf
		  ```
		- 这条配置的含义如下：
			- `*`：通配符，匹配仓库中所有文件；
			- `text=auto`：告诉 Git 这是文本文件（而非二进制），并自动判断：
				- 若文件仅包含可打印字符和常见控制字符（如换行、制表符），则视为文本；
				- 否则视为二进制文件，不进行行尾转换；
			- `eol=lf`：指定工作区中文本文件使用 LF（`\n`）作为行尾格式。
		- 这样配置后，Git 在处理换行时将遵循以下策略：
			- **提交时**：无论本地使用 CRLF 还是 LF，Git 都会统一转换为 LF 存入仓库；
			- **检出时**：Git 会将仓库中的 LF 保持为 LF 写入本地，不会被系统自动转换成 CRLF；
		- 这就避免了其他人提交了 CRLF 的行尾格式后，在你这边拉取时出现无意义的行尾改动。
	- `core.autocrlf` 的辅助作用
	  heading:: true
		- 有了 `.gitattributes` 中的 `eol=lf` 设置后，Git 的换行符处理完全由 `.gitattributes` 接管。而配置项 `core.autocrlf` 仅在未指定 `eol` 时生效。
		- Git 还提供了一个全局设置项 `core.autocrlf`，用于处理行尾转换：
			- ```bash
			  git config --global core.autocrlf input
			  ```
		- 参数说明：
			- `true`：提交时将 CRLF 转为 LF，检出时将 LF 转回 CRLF；
			- `input`：仅在提交时将 CRLF 转为 LF，检出时不转换；
			- `false`：提交和检出均不转换。
		- 在 `.gitattributes` 中已明确指定 `eol` 后，应将 `core.autocrlf` 设置为 `input` 或 `false`，以避免 Git 的两个自动换行符转换机制产生冲突。
-