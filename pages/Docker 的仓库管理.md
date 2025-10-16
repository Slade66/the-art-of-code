- **Registry 是什么？**
	- Registry（注册服务器）是一个“镜像中心”服务器，专门用来存储和分发各种 Docker 镜像（images）。
	- 就像 GitHub 是一个存放和分发代码仓库（code repositories）的中心一样，而 Docker Hub 则是最常用、最知名的那个公共 Registry。
- **Repository 是什么？**
	- Repository（仓库）是在 Registry 内部用来存放一组“相关联”镜像的文件夹。
	- 比如你有一个 Golang 项目，它的所有不同版本的镜像（像 `v1.0`、`v1.1` 或 `latest`）都会放在同一个仓库里。每个版本都用一个标签（Tag）来标记，这样你就知道哪个镜像对应哪个版本了。
	- **日常交流中的歧义说法**
		- 当说“我们公司用的是私有仓库”时，实际是指“我们公司使用的是一个私有的 Registry 服务（如自托管的 Harbor）”。在这种语境下，“仓库”被用作对 Registry 服务的泛指。
- **Public Registry 和 Private Registry**
	- **Public Registry**
		- 任何人都可以上传或拉取镜像。最广为人知的就是 Docker Hub。
		  id:: 6892f75a-1b68-47ce-9e30-958f3eed2743
		- **缺点**：
			- **安全风险**：任何人都可以上传，镜像可能包含漏洞或恶意代码。
			- **隐私问题**：不适合存放包含私有代码或敏感信息的商业项目镜像。
			- **速率限制**：匿名和免费用户在拉取镜像时有频率限制。
	- **Private Registry**
		- 只有授权用户才能访问、拉取或推送镜像。
		- Harbor 是 VMware 开源的一种常见的企业级自托管 Registry 解决方案。
		- Docker Registry 是 Docker 官方提供的基础镜像仓库实现，可直接以容器形式运行，部署便捷。它专注于镜像的存储与分发，不提供 Web UI、用户管理或漏洞扫描功能，管理操作主要依赖 API 调用或手动操作。
		- **优点**：
			- **安全性高**：可控的访问权限，有助于保护代码和知识产权；
			- **传输高效**：部署在内网，镜像拉取速度快，延迟低。
- **Registry 和 Repository 的关系树**
	- ```text
	  Registry (例如 Docker Hub)
	  │
	  ├── Repository (例如 library/ubuntu)
	  │   ├── Image (镜像) + Tag (标签: 20.04)
	  │   ├── Image (镜像) + Tag (标签: 22.04)
	  │   └── Image (镜像) + Tag (标签: latest)
	  │
	  └── Repository (例如 my-username/my-app)
	      ├── Image (镜像) + Tag (标签: v1.0)
	      └── Image (镜像) + Tag (标签: v1.1)
	  ```
- **镜像的名称格式**
	- 完整的镜像名称格式为：`[registry-host]/[repository-name]:[tag]`
	- `registry-host`：镜像仓库地址，默认为 Docker Hub（`docker.io`）；
	- `repository-name`：仓库名称，通常包含用户名或项目名，如 `bitnami/nginx`；
	- `tag`：用于标识镜像版本，如 `1.21.6`，默认值为 `latest`。
- ## 仓库管理命令
	- `docker login`
		- 在向镜像仓库推送或从私有仓库拉取镜像前，需要先登录。
		- ```shell
		  # 登录 Docker Hub（默认）
		  docker login
		  
		  # 登录指定的私有仓库（如 AWS ECR）
		  docker login -u AWS -p $(aws ecr get-login-password --region us-west-2) 123456789012.dkr.ecr.us-west-2.amazonaws.com
		  ```
		- 登录后，Docker 会将凭证加密存储在本地的 `~/.docker/config.json` 文件中。
	- `docker tag`
		- 给镜像打标签是推送前的重要步骤。`docker build` 创建的镜像默认不带 registry 地址，因此需要用 `docker tag` 创建一个符合目标仓库命名规范的新标签。
		- 格式：`docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`
		- ```shell
		  # 将本地镜像 my-app:1.0 标记为 Docker Hub 用户 a_user 下的镜像
		  docker tag my-app:1.0 a_user/my-app:1.0
		  
		  # 标记为私有仓库中的镜像
		  docker tag my-app:1.0 my-registry.com/my-project/my-app:1.0
		  ```
		- **镜像标签的最佳实践**
			- **结合 Git Commit SHA**
				- 为了实现高度可追溯性，建议在标签中加入 Git 的短 commit hash，使每个镜像都能精确对应到构建它的具体代码提交。
				- 示例：`my-app:v1.2.5-a1b2c3d`。
			- **采用语义化版本号**
				- 使用 `v<主版本>.<次版本>.<补丁版本>` 的格式，例如 `v1.2.5`，有助于清晰表达版本变更的级别。
			- **根据部署环境区分标签**
				- 在 CI/CD 流程中，可为不同环境打上明确的标签，如 `-staging`、`-prod` 等，方便识别和管理。
				- 示例：`my-app:v1.2.5-staging`、`my-app:v1.2.5-prod`。
	- `docker push`
		- 用于将本地镜像推送到远程仓库。
		- 推送前需确保已通过 `docker login` 登录，并使用 `docker tag` 正确标记镜像。
		- ```shell
		  docker push a_user/my-app:1.0
		  ```
	- `docker logout`
		- 用于登出已登录的镜像仓库，并删除本地保存的凭证信息。
		- ```shell
		  # 登出 Docker Hub
		  docker logout
		  
		  # 登出指定的私有仓库
		  docker logout my-registry.com
		  ```
	- `docker search`
		- 用于在 Docker Hub 上搜索可用的公共镜像。
		- ```shell
		  # 搜索与 "mysql" 相关的镜像
		  docker search mysql
		  ```
-