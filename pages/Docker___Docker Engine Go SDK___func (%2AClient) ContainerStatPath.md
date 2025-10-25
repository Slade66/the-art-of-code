- **作用：**
	- 该函数用于获取 Docker 容器文件系统内特定路径（文件或目录）的元数据。
	- 在不下载任何文件内容的前提下，通过一个 `HEAD` 请求，用最小的网络开销获取容器内某个文件或目录的元信息。
- **方法源码：**
	- ```go
	  func (cli *Client) ContainerStatPath(ctx context.Context, containerID, path string) (container.PathStat, error) {
	  	containerID, err := trimID("container", containerID)
	  	if err != nil {
	  		return container.PathStat{}, err
	  	}
	  
	  	query := url.Values{}
	  	query.Set("path", filepath.ToSlash(path)) // Normalize the paths used in the API.
	  
	  	resp, err := cli.head(ctx, "/containers/"+containerID+"/archive", query, nil)
	  	defer ensureReaderClosed(resp)
	  	if err != nil {
	  		return container.PathStat{}, err
	  	}
	  	return getContainerPathStatFromHeader(resp.Header)
	  }
	  ```
- **参数：**
	- `ctx context.Context`：上下文对象。用于控制请求的生命周期、超时和取消等。
	- `containerID string`：目标容器的 ID 或名称。
	- `path string`：容器内文件或目录的路径。
- **返回值：**
	- `container.PathStat`：路径的状态信息结构体（例如，文件类型、权限、大小、修改时间等）。如果 API 请求失败，返回空的 `PathStat` 结构。
	- `error`：如果函数执行过程中发生错误（如 ID 无效、网络错误、API 错误），则返回错误信息；否则返回 `nil`。
- **工作原理：**
	- **向 Docker API 发起请求**：它向 Docker 守护进程（Daemon）发起一个 HTTP `HEAD` 请求。
	  logseq.order-list-type:: number
		- **`HEAD` 请求的魔力**：HEAD 是一种特殊的 HTTP 方法，它告诉服务器：“我只需要响应头（Headers），不必发送实际的响应体（Body）。”
		  logseq.order-list-type:: number
		- **请求 URL**：请求的 API 路径是 `/containers/{containerID}/archive`。
		  logseq.order-list-type:: number
		- **请求参数**：它将您提供的 `path`（例如 `/app/config.json`）作为 URL 的查询参数 `path` 发送出去。
		  logseq.order-list-type:: number
			- 最终请求类似于：`HEAD /containers/123abc.../archive?path=%2Fapp%2Fconfig.json`
	- **Docker 守护进程的响应**：
	  logseq.order-list-type:: number
		- 当 Docker 守护进程收到这个 `HEAD` 请求，它会去容器的文件系统中查找您请求的 `path`。
		  logseq.order-list-type:: number
		- 它获取该路径的 `stat` 信息（包括：文件名、大小、权限模式、最后修改时间、是否为目录/文件/符号链接等）。
		  logseq.order-list-type:: number
		- 它不会像 `CopyFromContainer` 那样费力地将文件打包成 TAR 压缩包。
		  logseq.order-list-type:: number
		- 相反，它将获取到的 `stat` 信息序列化为 JSON 字符串，再进行 Base64 编码。
		  logseq.order-list-type:: number
		- 最后，它将这个编码后的字符串放进一个自定义的 HTTP 响应头中，名为 `X-Docker-Container-Path-Stat`。
		  logseq.order-list-type:: number
		- 它只返回这个响应头（以及其他标准 HTTP 头），响应体为空。
		  logseq.order-list-type:: number
	- **SDK 的返回值**：
	  logseq.order-list-type:: number
		- SDK 客户端（`cli.head(...)`）收到这个响应。
		  logseq.order-list-type:: number
		- 它立即调用 `getContainerPathStatFromHeader(resp.Header)`。这个辅助函数专门用于从响应头中提取 `X-Docker-Container-Path-Stat` 的值。
		  logseq.order-list-type:: number
		- 它解码 Base64 字符串，解析 JSON，并将其填充到一个 `container.PathStat` 结构体中。
		  logseq.order-list-type:: number
		- 最终，`ContainerStatPath` 将这个包含所有元信息的 `container.PathStat` 结构体返回给调用者。
		  logseq.order-list-type:: number
- **示例代码：**
	- ```go
	  package main
	  
	  import (
	  	"context"
	  	"fmt"
	  	"log"
	  	"time"
	  
	  	"github.com/docker/docker/client"
	  )
	  
	  func main() {
	  	// 1. 初始化 Docker 客户端
	  	cli, err := client.NewClientWithOpts(client.FromEnv, client.WithAPIVersionNegotiation())
	  	if err != nil {
	  		log.Fatalf("无法创建 Docker 客户端: %v", err)
	  	}
	  	// 在 main 函数结束时关闭客户端连接
	  	defer cli.Close()
	  
	  	// 创建一个上下文
	  	ctx := context.Background()
	  
	  	// --- 2. 调用 ContainerStatPath ---
	  
	  	// 容器ID或名称
	  	containerID := "apisix-quickstart"
	  
	  	// 我们要查询的容器内路径。/etc/hostname 是一个几乎所有容器都有的标准文件。
	  	pathInContainer := "/etc/hostname"
	  
	  	fmt.Printf("正在查询容器 '%s' 内的路径: '%s'\n", containerID, pathInContainer)
	  
	  	// 调用方法
	  	stat, err := cli.ContainerStatPath(ctx, containerID, pathInContainer)
	  	if err != nil {
	  		// 常见错误：容器不存在，或者路径不存在
	  		log.Fatalf("无法获取路径信息 (stat): %v", err)
	  	}
	  
	  	// 3. 打印结果
	  	fmt.Println("---------------------------------")
	  	fmt.Println("成功获取到元信息！")
	  	fmt.Printf("  路径名称 (Name): %s\n", stat.Name)
	  	fmt.Printf("  文件大小 (Size): %d 字节\n", stat.Size)
	  	fmt.Printf("  权限模式 (Mode): %s\n", stat.Mode)
	  	fmt.Printf("  修改时间 (Mtime): %s\n", stat.Mtime.Format(time.RFC3339))
	  	fmt.Printf("  是否为目录 (IsDir): %v\n", stat.Mode.IsDir())
	  	fmt.Println("---------------------------------")
	  }
	  
	  ```
- **注意：**
	- 它与 `CopyFromContainer` 访问的是同一个 API 终结点 (`/containers/{id}/archive`)，但使用了不同的 HTTP 方法。
-