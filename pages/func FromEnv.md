- **作用：**
	- 从操作系统环境变量中加载 Docker 客户端的配置（包括主机地址、TLS 认证和 API 版本）并应用到传入的 `*Client` 结构体上。
- **函数源码：**
	- ```go
	  func FromEnv(c *Client) error {
	  	ops := []Opt{
	  		WithTLSClientConfigFromEnv(),
	  		WithHostFromEnv(),
	  		WithVersionFromEnv(),
	  	}
	  	for _, op := range ops {
	  		if err := op(c); err != nil {
	  			return err
	  		}
	  	}
	  	return nil
	  }
	  ```
	- 它等同于同时调用了三个独立的配置函数：
		- WithTLSClientConfigFromEnv
		- WithHostFromEnv
		- WithVersionFromEnv
	- `FromEnv` 内部并没有直接读取环境变量，而是通过组合其他 `Opt` 函数来实现其功能，体现了函数选项模式的模块化优势。
- **参数：**
	- 一个指向 `Client` 结构体（Docker 客户端对象）的指针。这是函数执行配置的目标对象，函数会直接修改这个结构体的内部字段。
- **返回值：**
	- 如果客户端配置成功，返回 `nil`。
	- 如果在读取环境变量或应用配置时发生任何错误，则返回非 `nil` 的 `error`，表示客户端创建失败。
- **注意：**
	- `FromEnv` 依赖以下四个关键环境变量来配置客户端：
		- `DOCKER_HOST`：设置 Docker 守护进程的 URL 地址（例如 `unix:///var/run/docker.sock` 或 `tcp://192.168.1.10:2376`）。
		- `DOCKER_API_VERSION`：手动指定要使用的 Docker API 版本（例如 `1.40`）。如果设置，它会覆盖自动版本协商。留空则表示使用最新版本或启用协商。
		- `DOCKER_CERT_PATH`：指定包含 TLS 证书（`ca.pem`、`cert.pem`、`key.pem`）的目录路径，用于安全连接。
		- `DOCKER_TLS_VERIFY`：用于启用或禁用 TLS 证书验证。如果设置为任何非空值（如 `1`），则启用验证；如果为空，则默认不验证（不安全）。
-