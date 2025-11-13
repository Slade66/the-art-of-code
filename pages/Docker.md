- **什么是 Docker 镜像（Image）？**
	- Docker 镜像就像在虚拟机中安装好操作系统并配置好服务后生成的系统快照，用作创建容器的模板。
	- 它定义了容器运行所需的操作系统和应用环境，是构建容器的起点。
- Docker 命令
  heading:: true
	- `docker run`
	  collapsed:: true
		- 从镜像创建容器并立即运行。
		- **语法：**
			- ```bash
			  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
			  ```
		- `OPTIONS`
			- 可选参数，用于控制容器的运行行为。
			- `-d`：使容器在后台运行，不占用当前终端。
			- `--name <容器名>`：为容器指定名称，方便后续通过名称进行管理。
			- `-p <主机端口>:<容器端口>`：端口映射，将容器内的端口暴露给主机，从而通过主机端口访问容器中的服务。
			- `-v <本机目录>:<容器目录>`：目录挂载，将主机目录挂载到容器中，实现宿主机与容器之间的文件共享。
			- `--network <网络名>`：指定容器连接到的 Docker 网络，便于容器之间通信。
			  id:: 68f333e1-8f6e-41c6-8716-308ae95484fd
			- `-it`：默认情况下，容器启动后执行镜像中的命令，命令执行完毕容器即退出。加上 `-it` 后，容器将以交互模式运行，允许你像登录一台 Linux 主机一样进入容器，手动输入命令并实时查看输出。
			- `--rm`：容器退出后自动删除，适用于临时任务或测试场景，可避免系统中残留无用容器，相当于“用完即扔”。
		- `IMAGE`
			- 指定你希望运行的镜像。
			- 如果本地不存在该镜像，Docker 会自动从默认仓库 Docker Hub 下载。
	- `docker inspect`
	  collapsed:: true
		- **作用：**用于获取 Docker 对象（容器、镜像、网络、卷等）的详细配置信息，结果以 JSON 数组的格式返回。
		- **注意：**
			- 一定要将 `docker inspect` 的原始 JSON 输出通过管道 (`|`) 传递给 `jq` 工具进行语法高亮和过滤。
				- ```grep
				  docker inspect alert-nginx | jq '.[0].Mounts'
				  ```
				- 在 `jq '.[0].Mounts'` 中，第一个点号（`.`）代表 `jq` 中当前输入的整个 JSON 数据流（即 `docker inspect` 输出的数组）。
			- 使用 `-s` 或 `--size` 可以显示容器文件系统的大小（`SizeRw`）。
			- 您可以同时检查多个对象，它们的结果会包含在同一个 JSON 数组中。
		- **示例：**
			- ```bash
			  docker inspect -s my-container
			  docker inspect my-image:latest
			  ```
	- `docker network`
	  collapsed:: true
		- `docker network ls`：列出 Docker 宿主机上的所有网络。
		  collapsed:: true
			- ```bash
			  [root@kylin116 ~]# docker network ls
			  NETWORK ID     NAME                      DRIVER    SCOPE
			  8c993e58a249   1panel-network            bridge    local
			  af3c985c595a   bridge                    bridge    local
			  308816a43f91   example_default           bridge    local
			  59bc2f27f47b   host                      host      local
			  b7acf3b26487   none                      null      local
			  d11ae68c8128   poc_network               bridge    local
			  e223bda38aee   test-net                  bridge    local
			  1d55faa2a33e   victoriametrics_default   bridge    local
			  9ea9d7648078   zabbix-net                bridge    local
			  ```
		- `docker network create <网络名>`：创建新网络。
		  collapsed:: true
			- **服务发现**：在自定义网络中，容器可以通过容器名称互相访问，而无需使用 IP 地址（这在默认 `bridge` 网络中是不支持的）。
		- `docker network inspect <网络名>`：检查网络的配置、驱动和连接到该网络的容器。
		  collapsed:: true
			- ```bash
			  [root@kylin116 ~]# docker network inspect bridge
			  [
			      {
			          "Name": "bridge",
			          "Id": "af3c985c595a97e2dff2ef5c55171ff990c21d22704e6d0f7d7baa91fdf1962f",
			          "Created": "2025-09-21T17:24:53.976393033+08:00",
			          "Scope": "local",
			          "Driver": "bridge",
			          "EnableIPv6": false,
			          "IPAM": {
			              "Driver": "default",
			              "Options": null,
			              "Config": [
			                  {
			                      "Subnet": "172.17.0.0/16",
			                      "Gateway": "172.17.0.1"
			                  }
			              ]
			          },
			          "Internal": false,
			          "Attachable": false,
			          "Ingress": false,
			          "ConfigFrom": {
			              "Network": ""
			          },
			          "ConfigOnly": false,
			          "Containers": {
			              "12f339b0ccdecfca01f56cee020b88ebaf55e264961cfd08bb11e9048e66df87": {
			                  "Name": "mariadb-master",
			                  "EndpointID": "e0b62a02f30c8df5fad133f3db7d99bc49a8b88321c306aaea37a7d32c1b7570",
			                  "MacAddress": "02:42:ac:11:00:04",
			                  "IPv4Address": "172.17.0.4/16",
			                  "IPv6Address": ""
			              },
			              "25c93cb3b07f6d80b02e0c47b179a105fcb382e8dca0c62b2b8f3b1386b643a2": {
			                  "Name": "prometheus",
			                  "EndpointID": "6f091c2ff927d046b09e12f50f317147025fe75671f681867edd35b524eaa21e",
			                  "MacAddress": "02:42:ac:11:00:07",
			                  "IPv4Address": "172.17.0.7/16",
			                  "IPv6Address": ""
			              },
			              "30a4ddf04fb7c8bf08602f742963b7f8c29d351db6e0115830a8dba8542f2f92": {
			                  "Name": "etcd",
			                  "EndpointID": "c7815b596f81e6342bc4c472675457e96116f557ecf548e95857b28460c88dd6",
			                  "MacAddress": "02:42:ac:11:00:03",
			                  "IPv4Address": "172.17.0.3/16",
			                  "IPv6Address": ""
			              },
			              "a318005482ec6f9da66695b4862de2f8223462c2d0cdc46a67b63e1e1e895338": {
			                  "Name": "apisix-dashboard",
			                  "EndpointID": "32f33719fed6d0fd65e39cbb209002edc73c3ae542ec2c7be687ed5b677ba1b5",
			                  "MacAddress": "02:42:ac:11:00:06",
			                  "IPv4Address": "172.17.0.6/16",
			                  "IPv6Address": ""
			              },
			              "c723e19566927e2fbb6619e37cc91a9343eed8e8040cfc3adb2a1ebea84cb505": {
			                  "Name": "clickhouse-xiaoze-poc-test",
			                  "EndpointID": "fa17531bd4c0f9145c577a2c368a7ec0c06f2d8198c3813423f227bf548a61f6",
			                  "MacAddress": "02:42:ac:11:00:02",
			                  "IPv4Address": "172.17.0.2/16",
			                  "IPv6Address": ""
			              },
			              "ed9d560601c62272089cd9926bd4fd2f4e10dbf4f86e6280e53363ec61fff5a9": {
			                  "Name": "dpanel-plugin-explorer",
			                  "EndpointID": "834927d1385ad8eab2b3eea7dd0b6a22d917626b2075f7575c7c6294ed9a2ae5",
			                  "MacAddress": "02:42:ac:11:00:08",
			                  "IPv4Address": "172.17.0.8/16",
			                  "IPv6Address": ""
			              },
			              "fdb825996d734feb667cd5c12394d1c4db5d79124d7fdfb51891ac58c6440cd0": {
			                  "Name": "apisix",
			                  "EndpointID": "eb245462b88b49cfa7159054e491e70083506b93da90b0a90fc3add90917f1d5",
			                  "MacAddress": "02:42:ac:11:00:05",
			                  "IPv4Address": "172.17.0.5/16",
			                  "IPv6Address": ""
			              }
			          },
			          "Options": {
			              "com.docker.network.bridge.default_bridge": "true",
			              "com.docker.network.bridge.enable_icc": "true",
			              "com.docker.network.bridge.enable_ip_masquerade": "true",
			              "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
			              "com.docker.network.bridge.name": "docker0",
			              "com.docker.network.driver.mtu": "1500"
			          },
			          "Labels": {}
			      }
			  ]
			  ```
		- `docker network connect/disconnect <NET> <CONTAINER>`：把容器连接或断开到网络。
		- `docker network rm <网络名>`：清理不再使用的网络。
		  collapsed:: true
			- 注意：只有在没有容器连接时才能删除。
	- `docker login`
	  collapsed:: true
		- 作用：让你登录到一个指定的 Docker 镜像仓库（Registry），从而获得推送（push）和拉取（pull）私有镜像的权限。
		- 镜像仓库就像存放代码的仓库（比如 GitHub），不过它存放的是打包好的 Docker 镜像。最常见的镜像仓库：Docker Hub，它是 Docker 官方的公共仓库。
		- **拉取公共镜像**：通常不需要登录，任何人都可以匿名拉取。
		- **推送任何镜像**：**必须登录**。你总得告诉仓库你是谁，有没有权限把镜像上传到这个地方。
		- **拉取私有镜像**：**必须登录**。私有镜像是为了保护代码和应用，只有经过授权的用户才能拉取。
		- 你工作中 99% 的场景都是将你的应用（比如你用 Kratos 写的微服务）打包成镜像，然后推送到公司的**私有仓库**中，再由 Kubernetes 或其他部署系统从私有仓库拉取镜像来运行。这个过程的第一步就是 `docker login`。
		- 登录 Docker Hub
			- 省略 `SERVER` 则登录 Docker Hub。
			- docker login
			- 执行后，终端会提示你输入用户名和密码：
		- 登录私有仓库
			- docker login my-registry.company.com:5000
		- 登录信息存放在哪里？
			- 当你成功执行 `docker login` 后，Docker 会将认证信息（一个经过编码的授权令牌）保存在你用户主目录下的一个文件里：
			- **文件路径**: `~/.docker/config.json`
			- 它是一个 JSON 格式的文件，`auths` 字段下记录了你登录过的所有仓库地址和对应的授权令牌。
	- `docker logout`
	  collapsed:: true
		- 如果你想清除登录凭据，可以使用 `docker logout` 命令
		- 登出 Docker Hub
		  docker logout
		- 登出指定的私有仓库
		  docker logout my-registry.company.com:5000
	-
- [[Docker 的仓库管理]]
- [[Docker 的镜像管理]]
- [[容器（Container）]]
- [[Docker Desktop]]
- [[Docker/Docker Engine Go SDK]]
- [[Docker/network]]
- ## 安装和配置 Docker
  collapsed:: true
	- **安装 Docker**
	  collapsed:: true
		- **先配置好 Linux 的网络代理。**
		  logseq.order-list-type:: number
		- **跟着做：**https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
		  logseq.order-list-type:: number
	- **配置 Docker 的网络代理**
	  collapsed:: true
		- `mkdir -p /etc/systemd/system/docker.service.d`
		  logseq.order-list-type:: number
		- `vim /etc/systemd/system/docker.service.d/http-proxy.conf`
		  logseq.order-list-type:: number
			- ```bash
			  [Service]
			  Environment="HTTP_PROXY=http://127.0.0.1:7897"
			  Environment="HTTPS_PROXY=http://127.0.0.1:7897"
			  Environment="NO_PROXY=localhost,127.0.0.1,::1,*.local,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
			  ```
		- `systemctl daemon-reload && sudo systemctl restart docker`
		  logseq.order-list-type:: number
		- `docker info | grep -i proxy` 应显示：
		  logseq.order-list-type:: number
			- ```bash
			  root@liyuze:~# docker info | grep -i proxy
			   HTTP Proxy: http://127.0.0.1:7897
			   HTTPS Proxy: http://127.0.0.1:7897
			   No Proxy: localhost,127.0.0.1,::1,*.local,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16
			  ```
	- **开启 Docker 的远程访问**
	  collapsed:: true
		- `vim /etc/systemd/system/docker.service.d/override.conf`
		  logseq.order-list-type:: number
		  collapsed:: true
			- ```bash
			  [Service]
			  ExecStart=
			  ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375
			  ```
		- `systemctl daemon-reload && sudo systemctl restart docker`
		  logseq.order-list-type:: number
		- `ss -lntp | grep 2375`
		  logseq.order-list-type:: number
- ## 注意
  collapsed:: true
	- **不要用镜像的 latest 标签，因为这个标签会变动，导致使用镜像的命令失效。**
	  collapsed:: true
		- nacos 2.4 的启动命令和 3+ 版本的启动参数不一样，会出现启动 nacos 的命令以前能用，过段时间就失效了，因为 latest 标签指向了 3+。
	- **把依赖的镜像找个镜像仓库存着，不然哪天这个镜像就被开发者删了，导致脚本执行错误。**
	  collapsed:: true
		- bitnami 将 etcd 的镜像移动到另一个仓库，导致 APISIX 官方提供的容器启动命令执行失败。
	- [[连接不上 Docker 中的 MySQL 服务]]
	- **在容器中访问宿主机或其他容器时，不能使用 `localhost` 或 `127.0.0.1`。**
	  collapsed:: true
		- 在计算机网络中，`localhost`（或 `127.0.0.1`）表示“当前这台机器”。
		- 当程序运行在 Docker 容器中时，这个容器就像一台独立的“小电脑”。如果在容器内使用 `localhost` 或 `127.0.0.1` 访问宿主机端口，实际上是让容器去找“自己”的端口，而不是宿主机的端口。由于容器内部并没有对应的服务，结果自然会出现“连接被拒绝”。
		- **解决方案：**
			- Docker 提供了一个特殊的主机名 `host.docker.internal`，它会自动解析为宿主机的内部 IP 地址，可用于容器访问宿主机服务。
			- 对于同一 Docker 网络下的多个容器，它们可以通过“容器名”互相访问。
-