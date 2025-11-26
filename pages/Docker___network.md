## 默认的桥接网络
collapsed:: true
	- 当 Linux 上的 Docker Engine 首次启动时，会创建一个名为 “bridge” 的默认网络。当你运行容器而没有显式指定网络驱动（未指定 `--network` 选项）时，容器会自动连接到这个默认桥接网络。
	- 连接到默认桥接网络的容器可以访问 Docker 主机外部的网络服务。如果 Docker 主机能够访问互联网，那么容器无需额外配置即可直接访问互联网。
	- 连接到同一个 bridge 网络的容器之间可以互相通信；外部（即主机以外）无法直接访问这个网络里的容器。
- ## bridge 模式下的容器端口
  collapsed:: true
	- 同一个 bridge 网络内的容器和 Docker 主机可以访问容器端口（即使未发布）；但不同网络中的容器和主机外部的程序不能访问容器的端口。
	- 必须使用 `--publish` 或 `-p` 参数发布端口：`-p [宿主机IP:]宿主机端口:容器端口`
		- 把宿主机的端口映射到容器内部的端口，外部访问宿主机的端口，实际上就会被转发到容器的端口上。
		- Docker 主机充当中间代理，外部访问主机端口后，Docker 自动把流量转发到容器对应端口。
- ## 自定义网络
  collapsed:: true
	- 用户通过 `docker network create` 命令创建的网络。
	- 在默认配置下，连接到默认桥接网络的容器可以通过容器 IP 地址相互进行不受限制的网络访问，但无法通过容器名称相互访问。
	- 你可以创建自定义的用户定义网络，并将多组容器连接到同一个网络。一旦连接到用户定义网络，容器之间就可以通过容器 IP 地址或容器名称进行通信。
- ## 网络驱动
  collapsed:: true
	- **总结一句话理解：**Docker 的网络驱动决定了容器“怎么连网”，你可以选择虚拟交换机（bridge）、共享宿主机（host）、组建跨主机隧道（overlay）、甚至让容器直接变成“网络上的独立电脑”（macvlan/ipvlan）。
	- bridge：默认的网络驱动。同一台主机上的多个容器之间可以互相通信。给每个容器一根“网线”，插在 Docker 虚拟交换机上
	- host：移除容器与宿主机之间的网络隔离，让容器直接使用宿主机的网络。
	- none：拔掉网线，完全隔离容器。将容器与主机及其他容器完全隔离，使它们无法通信。无需网络的任务（例如纯计算任务）
	- overlay：连接多台 Docker 守护进程（daemons），让容器可以跨主机通信。多主机部署的分布式系统
	- ipvlan：多个容器共用一张网卡但不同 IP。
	- macvlan：给每个容器一块独立网卡（分配一个独立的 MAC 地址），使容器直接出现在局域网里，让它在网络中表现得像一台独立的物理设备。
- ## 把容器连接到多个网络
  collapsed:: true
	- 一个容器可以连接到多个 Docker 网络，就像一台电脑插多根网线。
	- 创建容器时，可以多次传入 `--network` 参数。
	- 在容器已经运行时，用 `docker network connect` 命令。
- ## Docker 网络的自动分配机制
  collapsed:: true
	- Docker 在创建网络时会自动：
		- 为网络分配一个子网（例如 `172.18.0.0/16`）；
		- 配置好默认网关（一般是 `.1`），容器获得的 IP 从 `.2` 开始。
	- 容器在加入每一个 Docker 网络时，都会获得一个 IP 地址。你不需要手动给每个容器配 IP，Docker 会像 DHCP（自动分配 IP 的服务）那样帮你管理。
	- 可以通过 `--ip` 或 `--ip6` 参数，为该容器在特定网络中指定固定 IP。
- ## DNS 服务
  collapsed:: true
	- 当容器连接到默认的 bridge 网络时，它会直接获得宿主机 `/etc/resolv.conf` 的一个副本，继承宿主机的 DNS 配置。
	- 当容器连接到自定义网络时，它会使用 Docker 自带的内置 DNS 服务器。这个内置 DNS 服务器主要有两个作用：
		- **内部服务解析**：支持容器之间通过**容器名**互相访问。
			- 例如：`ping web` 就能解析到名为 `web` 的容器 IP。
		- **外部域名转发**：当容器访问外部域名时，Docker 会把查询转发给宿主机配置的 DNS 服务器。
	- `--dns`：让容器使用你指定的 DNS 服务器，而不再使用宿主机默认配置。
	  id:: 69158b16-3348-48cd-999a-27592b94afb3
- ## 容器主机名
  collapsed:: true
	- 默认情况下，Docker 会把容器的 **ID**（那串十六进制字符）作为主机名。
	- `--hostname`：设置容器的主机名。
	- 这样在容器内部执行 `hostname` 命令和查看 `/etc/hosts` 就会显示自定义的主机名。
- ## 容器的 /etc/hosts
  collapsed:: true
	- 在每个容器中，Docker 会自动生成 `/etc/hosts` 文件：
		- ```bash
		  root@7fb63e51c36b:/# cat /etc/hosts
		  127.0.0.1       localhost
		  ::1     localhost ip6-localhost ip6-loopback
		  fe00::  ip6-localnet
		  ff00::  ip6-mcastprefix
		  ff02::1 ip6-allnodes
		  ff02::2 ip6-allrouters
		  172.17.0.2      7fb63e51c36b
		  ```
	- 宿主机上的 `/etc/hosts` 文件不会自动复制进容器。
	- 如果你希望容器能识别额外的主机名，就需要通过 `--add-host` 参数手动指定，这样，Docker 会在容器的 `/etc/hosts` 文件中添加该条目。
- ## container 网络模式
  collapsed:: true
	- `--network container:<容器名称或ID>`
	- 当前容器不会创建新的网络接口，不拥有自己的 IP 地址，直接复用另一个容器的网络栈，简单说，就是：“我不自己搞网卡了，直接用你那张网卡。”
	- **使用场景：**
		- 调试容器网络：
			- ```bash
			  docker run -it --rm --network container:web busybox sh
			  ```
			- 例如你怀疑某个容器的网络有问题，可以新建一个“诊断容器”共享它的网络，然后在里面用 `ping`, `nslookup`, `curl` 等命令测试网络。
- ## Overlay
  collapsed:: true
	- **使容器能跨主机通信：**
		- 普通的 bridge 网络只能让同一主机上的容器通信；Overlay 网络则像在多台主机上“铺了一层虚拟的交换机”，不管容器在哪台机器上，只要连到这个网络里，就能像在同一个局域网中一样通信。
		- Overlay 网络驱动会在多个 Docker 守护进程所在的主机之间，创建一个“分布式网络”。这个网络“覆盖”（overlay）在每台主机自己的本地网络之上，使得连接到该网络的容器，即使分布在不同主机上，也能相互通信。
	- 要实现 Overlay 网络通信，Docker 要求处于 Swarm 模式。
	- Overlay 网络内的容器可以直接通过 Docker 内置的 DNS 服务 相互通信。也就是说它们可以通过容器名互相访问。
	- **注意：**
		- overlay 只能访问对方的 overlay IP，不可能访问对方的 bridge IP。
		- 在分布式场景下：所有要互通的容器，都应该共用同一个 overlay 网络。
	- **场景：**
		- 分布式服务通信。
		- 想象一下，你有几台电脑（不同主机），每台电脑上都运行着几个 Docker 容器。如果这些容器要“像在同一个局域网里一样”相互通信，就需要 **Overlay 网络**。
		- Overlay 网络就像在不同电脑之间铺设了一条“虚拟的网线隧道”。虽然它们物理上不在同一个网络中，但通过这条虚拟隧道，容器之间的通信可以跨越主机边界，就像大家都插在同一个交换机上一样。
	- Docker 在底层会自动完成：
		- 建立隧道：Overlay 网络本质上是通过 VXLAN 协议 在主机之间建立“虚拟隧道”，传输容器的网络包。
		- 管理各主机的路由；
		- 把发往其他主机的包封装、加密、再转发；
		- 解封包后送到正确的容器。
	- **配置（多主机独立容器通信（非 Swarm 服务））：**
		- **每个主机上需要开放的端口：**
			- **为什么要开放这些端口？**
				- 不同主机上的 Docker 守护进程需要互相交换信息、建立隧道、同步容器的网络状态，因此必须开放特定端口。
			- `2377/tcp`：Swarm 控制平面的默认端口
			- `4789/udp`：Overlay 网络数据传输的默认端口，真正传输数据包
			- `7946/tcp`, `7946/udp`：节点之间通信所使用的端口，用于节点相互发现与同步
		- **在机器 A 上初始化 Swarm：**
			- ```bash
			  docker swarm init
			  ```
		- **将机器 B 加入 Swarm：**
			- ```bash
			  docker swarm join --token <your_token> <your_ip_address>:2377
			  ```
			- 在机器 B 上执行上一步输出的 join 命令，将其作为 worker 节点加入 Swarm。
		- **机器 A 创建一个 attachable 的 overlay 网络：**
			- ```bash
			  docker network create -d overlay --attachable my-attachable-overlay
			  ```
			- `-d overlay` 表示使用 Overlay 驱动；
			- `--attachable` 默认情况下，Overlay 网络只给 Swarm 服务 使用。这个参数表示该网络允许独立容器和 Swarm 服务 都能连接上； 如果没有这个选项，那么只有 Swarm 服务才能使用这个网络。
		- **把机器 B 加入机器 A 创建的 overlay 网络：**
			- 将容器加入到 Overlay 网络后，它就能和该网络中的其他容器通信，而无需在各 Docker 主机上手动配置路由。前提是这些主机已经加入同一个 Swarm。
	- **加密 Overlay 网络通信：**
	  collapsed:: true
		- 使用 `--opt encrypted` 参数来加密在 Overlay 网络中传输的应用数据。
		- 这会在 VXLAN（Virtual Extensible LAN）层启用 IPsec 加密。
		- 这种加密机制会带来一定的性能损耗，在正式生产环境使用前先测试性能。
		- 不要让 Windows 容器加入加密的 Overlay 网络。因为 Overlay 网络加密功能在 Windows 上不受支持。Swarm 虽然不会直接报错，但会出现以下问题：
			- Windows 容器无法与 Linux 容器通信
			- Windows 容器之间的流量不会被加密
-