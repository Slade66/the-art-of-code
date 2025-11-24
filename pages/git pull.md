- **作用：**
	- `git pull` 会去拉取你当前所在分支对应的远程分支的最新提交，并把它合并到当前分支。
		- pull 的本质 = fetch（当前分支） + merge
	- 如果你在 `main` 分支上：`git pull`
		- 等价于：
			- ```bash
			  git fetch origin main
			  git merge origin/main
			  ```
- **注意：**
	- **`git pull` 不会去查看其它分支的变化**
		- 不会发现远程新增了什么分支——新增远程分支看不到。
		- 不会发现远程删掉了什么分支——旧远程分支还残留。
		- 这些属于“远程分支列表（refs）过期”的问题。
		- 这种情况需要：`git fetch --prune`，这会把新分支信息拿下来并删除已删除的旧分支记录。
-