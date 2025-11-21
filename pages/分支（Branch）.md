-
- ### 分支有什么用？
	- **并行开发**：团队成员可以在各自的分支上开发，互不干扰。
	- **保证主干稳定**：在不影响主分支稳定性的前提下，你可以开发新功能。所有开发工作都在分支上进行，只有功能开发完成并通过测试后，才会合并到主分支，从而确保主分支始终保持可用状态。
- ### 分支的原理
	- 分支只是一个指针。在 Git 中，一个分支（Branch）本质上只是一个指向某个提交（Commit）的轻量级可移动的指针。
- ### 分支命令
	- **查看分支：**
		- `git branch`：查看所有本地分支。
		- `git branch -r`：查看所有远程分支。
		- `git branch -a`：查看所有本地和远程分支。
		- `git branch -v`：查看分支及其最近一次提交的信息。
	- **创建分支：**
		- `git branch [分支名]`：创建一个新分支，但当前仍停留在原分支上。
	- **切换分支：**
		- `git checkout [分支名]`：切换到已有分支。
		- `git switch [分支名]`：同上，现在更推荐的方式。
	- **创建并立即切换分支：**
		- `git checkout -b [分支名]`：创建新分支并立即切换过去。
		- `git switch -c [分支名]`：同上，现在更推荐的方式。
	- **合并分支：**
		- ```bash
		  # 切换回目标分支（例如 main）
		  git switch main
		  
		  # 拉取远程仓库的最新更新，确保本地 main 是最新的
		  git pull origin main
		  
		  # 将 feature/register 分支的修改合并到当前所在分支
		  git merge feature/register
		  ```
		- **解决合并冲突：**
			- 当 Git 无法自动合并两个分支的修改（例如同一文件的同一行被多人修改）时，就会产生冲突。
			- **处理步骤：**
				- 手动打开冲突的文件，解决冲突。
				  logseq.order-list-type:: number
				- 解决后，使用 `git add <文件名>` 告诉 Git 冲突已解决。
				  logseq.order-list-type:: number
				- 最后使用 `git commit` 来完成这次合并。
				  logseq.order-list-type:: number
	- **删除分支：**
		- `git branch -d [分支名]`：删除本地已合并的分支（安全删除）。如果该分支还有未合并的提交，命令会失败。
		- `git branch -D [分支名]`：强制删除本地分支，无论该分支的改动是否已合并，都会被直接删除。
	- **远程分支协作：**
		- **推送本地分支到远程：**`git push -u origin [远程分支名]`
			- 当你第一次在本地创建新分支时，远程仓库还不知道它的存在，需要通过推送将其上传。
			- `-u` 或 `--set-upstream` 参数：
				- 用于建立本地分支与远程分支的追踪关系。
				- 建立追踪关系后，以后在该分支上执行 `git push` 或 `git pull` 时，无需再指定远程仓库和分支名，Git 会自动把操作应用到对应的远程分支。
		- **拉取远程分支的更新：**`git pull`
			- `git pull` 实际上是两个命令的组合：
				- `git fetch`：从远程仓库下载最新的改动到本地，但不会自动合并到你当前的工作分支。
				- `git merge`：将远程分支的更新合并到当前本地分支。
				- ```bash
				  # git pull = git fetch + git merge
				  git pull origin main
				  # 等价于：
				  git fetch origin main  # 1. 从远程拉取 main 分支的最新代码
				  git merge origin/main  # 2. 将远程 main 分支合并到当前分支
				  ```
		- **删除远程分支：**`git push origin --delete [远程分支名]`
		- **拉取远程分支：**
			- `git switch <远程分支>` 或 `git checkout <远程分支>`
				- **注意：**
					- 当你执行 `git switch develop` 或 `git checkout develop` 时（假设本地不存在 `develop` 分支），Git 并不是直接“切换”到远程分支，因为你不能直接在远程跟踪分支（如 `origin/develop`）上工作。远程跟踪分支是你本地仓库对远程仓库状态的一个“只读”快照。
				- **工作流程：**
					- 当你输入 `git switch develop` 时，Git 首先会检查：“本地是否存在一个叫做 `develop` 的分支？”
					- 如果有，就直接切过去。
					- 如果没有，它会继续检查：“是否存在一个唯一的远程跟踪分支，其名字在去掉远程仓库名（如 `origin/`）后，恰好是 `develop`？”
						- **注意：**
							- 如果你有多个远程仓库（比如 `origin` 和 `upstream`），并且它们都有 `develop` 分支（即存在 `origin/develop` 和 `upstream/develop`），Git 会提示你命令有歧义，你需要更明确地指定。
					- 一旦 Git 找到了唯一的远程跟踪分支 `origin/develop`，它就会在本地创建一个新的分支，也命名为 `develop`。
					- Git 会自动设置新创建的本地 `develop` 分支去“跟踪”远程的 `origin/develop` 分支。
						- 这个“跟踪关系”意味着：
							- 当你在这个分支上使用 `git pull` 时，Git 知道要去 `origin` 拉取 `develop` 分支的更新。
							- 当你使用 `git push` 时，Git 知道要把你的提交推送到 `origin` 的 `develop` 分支上。
					- 最后一步，Git 会将你的工作区切换到这个刚刚创建的本地 `develop` 分支上。
			- **指定本地分支名和它要跟踪的远程分支：**
				- ```bash
				  git checkout -b dev origin/develop
				  ```
				- 在本地创建一个名为 `dev` 的新分支。这个新分支将以远程的 `origin/develop` 分支为基础，并自动设置跟踪关系。
				- 这个方法在你想要用一个不同的本地分支名来跟踪远程分支时特别有用。
- [[git fetch]]
- **不小心提交到了本地的 main 分支，但还没有推送到远程怎么办？**
	- 基于当前的 `main` 创建一个新的分支：`git switch -c my-role-features`
		- 这个命令创建了一个名为 `my-role-features` 的新指针。这个新指针也指向 `main` 当前所指向的提交。
		- 现在你的新提交有了两个分支指针指向它们：`main` 和 `my-role-features`。这确保了你的工作不会丢失。
	- 先切换回 `main` 分支，然后重置本地 `main` 到远程状态：
		- ```bash
		  git switch main
		  git reset --hard origin/main
		  ```
		- 它会把本地 `main` 分支回退到远程分支 `origin/main` 的位置。
		- **前提：**必须先切换回 `main` 分支，因为 `git reset` 总是移动当前所在分支的指针。
	- 这种操作模式的原理是：利用新分支来“缓存”你想保留的提交历史，然后利用 `git reset --hard` 来强制性地、安全地重置一个分支的指针和工作区。
-