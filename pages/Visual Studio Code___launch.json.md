- VS Code 的调试配置文件 launch.json，用来告诉 VS Code：当你点击“Run and Debug”时，应当以什么方式启动调试器来调试你的程序。
- **示例：**
	- ```go
	  {
	    "version": "0.2.0",
	    "configurations": [
	      {
	        "name": "Debug docker-panel",
	        "type": "go",
	        "request": "launch",
	        "mode": "debug",
	        "program": "${workspaceFolder}/cmd/docker-panel",
	        "cwd": "${workspaceFolder}",
	        "env": {
	          "DEBUG": "true"
	        }
	      }
	    ]
	  }
	  ```
	- `"version": "0.2.0"`：VS Code 调试配置文件的版本号，目前官方给出的模板版本就是 `0.2.0`。
	- `"configurations": [...]`：一个数组，用来定义多个调试方案；数组中的每个对象都表示一个独立的调试入口。
	- `"name": "Debug docker-panel"`：该调试配置在 VS Code 调试面板中显示的名称，方便选择并启动。
	- `"type": "go"`：告诉 VS Code 这是一个 Go 语言的调试任务，会使用 Go 扩展进行调试（底层基于 Delve）。
	- `"request": "launch"`：表示由 VS Code 负责启动被调试的程序；另一种常见取值是 `"attach"`，用于附加到已经运行的进程。
	- `"mode": "debug"`：指定调试模式。`debug` 表示使用 Delve 的 DAP 模式来编译并启动可执行文件，支持断点、单步执行等调试功能。
	- `"program": "${workspaceFolder}/cmd/docker-panel"`：指定要调试的程序入口，决定具体调试哪个包或可执行文件。
		- `${workspaceFolder}` 会被替换为当前工作区的根目录。
	- `"cwd": "${workspaceFolder}"`：current working directory，表示调试进程启动时的当前工作目录，决定程序是“从哪个目录”开始运行。
		- 在 VS Code 的 Go 调试中，默认的工作目录通常是 `program` 指向的目录，例如 `${workspaceFolder}/cmd/network-config-generator`。
	- `"env"`：以键值对的形式设置环境变量。
	- `envFile`：如果你有大量的环境变量，或想在代码库中共享环境配置，可以使用 `envFile` 属性指定一个 `.env` 文件来加载变量。
	  collapsed:: true
		- 首先，在你的项目根目录下创建一个文件（例如，命名为 `.env`）：
			- ```bash
			  DOCKER_HOST=tcp://192.168.1.5:2375
			  DOCKER_API_VERSION=1.40
			  ```
		- 然后，在 `launch.json` 配置中引用它：
			- ```json
			  {
			      "version": "0.2.0",
			      "configurations": [
			          {
			              "name": "Debug with .env file",
			              "type": "go",
			              "request": "launch",
			              "mode": "debug",
			              "program": "${workspaceFolder}/cmd/docker-panel",
			              "cwd": "${workspaceFolder}",
			              "envFile": "${workspaceFolder}/.env"
			          }
			      ]
			  }
			  ```
	- `args`：命令行参数列表。命令行启动程序时传进去的参数。
-