-
- ## launch.json
	- VS Code 的调试配置文件 `launch.json`，用于告诉 VS Code：当你点击“Run and Debug”时，要如何启动调试器去调试你的程序。
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
		        "cwd": "${workspaceFolder}"
		      }
		    ]
		  }
		  ```
		- `"version": "0.2.0"`：VS Code 调试配置文件的版本号，目前官方模板就是 `0.2.0`。
		- `"configurations": [...]`：一个数组。你可以在里面放多个调试方案，每个对象表示一个独立的调试入口。
		- `"name": "Debug docker-panel"`：这条配置在 VS Code 调试面板里显示的名字，方便选中后启动。
		- `"type": "go"`：告诉 VS Code 这是 Go 语言的调试任务，会调用 Go 扩展（底层使用 Delve）。
		- `"request": "launch"`：表示“由 VS Code 负责启动被调试程序”。另一种常见写法是 `"attach"`（附加到已有进程）。
		- `"mode": "debug"`：指定调试模式。`debug` 会让 VS Code 使用 Delve 的 DAP 模式编译并启动可执行文件，支持断点、单步等功能。
		- `"program": "${workspaceFolder}/cmd/docker-panel"`：指向要调试的程序入口。决定“调哪个包/可执行文件”。
			- `${workspaceFolder}` 会被替换成当前工作区根目录，这里相当于运行 `go build ./cmd/docker-panel` 再调试。
		- `"cwd": "${workspaceFolder}"`：**current working directory**。运行调试进程时的当前工作目录。决定“从哪个目录启动它”。
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
	- **最小调试运行示例：**
		- ```json
		  {
		      "version": "0.2.0",
		      "configurations": [
		          {
		              "name": "Launch User Service",
		              "type": "go",
		              "request": "launch",
		              "mode": "debug",
		              "program": "${workspaceFolder}/cmd/user-service"
		          }
		      ]
		  }
		  ```
-