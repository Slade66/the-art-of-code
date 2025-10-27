- **作用：**查看文件的尾部内容。
- **默认用法：查看最后 10 行**
	- 这是最基本的操作，当你只想快速看一眼文件的末尾（比如看一个日志文件最后发生了什么）时使用。
	- ```bash
	  # 查看 filename.log 的最后 10 行
	  tail filename.log
	  ```
- **-n：指定行数**
	- 默认的 10 行通常不够用。`-n` 允许你指定到底要看多少行。
	- ```bash
	  # 查看最后 20 行
	  tail -n 20 filename.log
	  
	  # -n 也可以省略，直接写 -[数字] 效果一样
	  tail -20 filename.log
	  ```
- **-n +[数字]：从第 N 行开始看，一直显示到文件末尾。**
	- ```bash
	  # 从第 5 行开始显示，直到文件结束
	  tail -n +5 filename.log
	  ```
- **-f / -F：实时监控文件**
	- 当文件有新内容追加时，会实时打印出来。
	- `tail -f filename.log`
		- 它实际上跟踪的是“文件描述符”。如果日志系统进行了“日志轮替”，例如将 access.log 重命名为 access.log.1 并重新创建一个新的 access.log，那么 `tail -f` 仍会继续跟踪原来的、已重命名为 access.log.1 的旧文件，而不是新的 access.log。
	- `tail -F filename.log`
		- 它跟踪的是“文件名”。当发现 `access.log` 被重命名或删除时，会持续重试；一旦新的 `access.log` 文件被创建，就会自动切换并继续监控新文件。
-