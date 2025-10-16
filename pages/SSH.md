-
- SSH 服务配置
  heading:: true
	- 通过 SSH 密钥进行免密登录
	  heading:: true
		- 如果在某个用户的 `.ssh` 目录下创建了 `authorized_keys` 文件并将公钥添加其中，那么之后可以使用对应的私钥进行该用户的免密登录，SSH 服务将通过公钥进行验证。
		- **步骤：**
			- 在开发机上生成 SSH 密钥对，得到私钥和公钥。
			  logseq.order-list-type:: number
			- 将公钥文件内容添加到目标服务器上需要进行免密登录的用户的 `~/.ssh/authorized_keys` 文件中。
			  logseq.order-list-type:: number
				- **方式一：**
					- ```bash
					  echo "你的公钥内容" >> ~/.ssh/authorized_keys
					  ```
					- 请注意，这里使用了 `>>` 来追加内容，而不是 `>`，因为 `>` 会覆盖文件内容。
				- **方式二：**
					- `ssh-copy-id` 是一个便捷的命令行工具，用于将本地 SSH 公钥自动复制到远程服务器指定用户的 `~/.ssh/authorized_keys` 文件中，从而实现无密码登录。它能自动完成整个过程，包括创建 `~/.ssh` 目录和设置文件权限，避免了手动复制和权限配置的繁琐步骤。
					- ```bash
					  ssh-copy-id user@remote_host
					  ```
			- **在开发机上通过 SSH 连接目标服务器的用户：**
			  logseq.order-list-type:: number
				- ```bash
				  ssh user1@your_server_ip
				  ```
				- 如果配置正确，会直接登录而无需输入密码。
		- **注意：**
			- 如果你希望为 root 用户配置 SSH 密钥登录，需将公钥添加到 `root` 用户的 `~/.ssh/authorized_keys` 文件中。
			- 如果 `~/.ssh/` 目录下没有 `authorized_keys` 文件，可以手动创建该文件。
			- **设置正确的文件权限：**
				- 在进行公钥认证时，SSH 服务会检查用户的 `~/.ssh` 目录及其中的 `authorized_keys` 文件权限。为了确保安全，SSH 要求这两个文件的权限设置必须严格。如果权限设置不当，其他用户可能会访问该目录、获取密钥或篡改配置，从而带来安全风险。若权限不符合要求，SSH 服务会拒绝认证请求，认为文件可能已被篡改或存在安全隐患。
				- ```bash
				  chmod 700 ~/.ssh
				  chmod 600 ~/.ssh/authorized_keys
				  ```
	- **配置 SSH 允许使用密码登录**
		- **修改 SSH 配置文件**
			- ```bash
			  sudo vim /etc/ssh/sshd_config
			  ```
		- **启用密码登录**
			- 找到 `PasswordAuthentication` 参数，并设置为 `yes`。
			- 该参数是全局开关，若设置为 `no`，则所有用户（包括 `root`）都无法使用密码登录。如果该行以 `#` 开头，表示被注释，请去掉 `#` 才能生效。
		- **允许 root 用户通过密码登录**
			- 找到 `PermitRootLogin` 参数，并设置为 `yes`。
			- `prohibit-password` 或 `without-password`：禁止使用密码登录，只允许密钥等方式。
			- `yes`：允许使用任何方式登录（包括密码登录）。
-