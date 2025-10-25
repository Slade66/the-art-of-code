- **作用：**
	- 该函数用于将 Docker 容器文件系统内特定路径（文件或目录）的内容复制到主机上。
	- 它通过 Docker Engine API 获取内容，并将其封装在一个 TAR 归档格式的流中返回给调用者，同时返回该源路径的元数据。
- **方法源码：**
	- ```go
	  func (cli *Client) CopyFromContainer(ctx context.Context, containerID, srcPath string) (io.ReadCloser, container.PathStat, error) {
	  	containerID, err := trimID("container", containerID)
	  	if err != nil {
	  		return nil, container.PathStat{}, err
	  	}
	  
	  	query := make(url.Values, 1)
	  	query.Set("path", filepath.ToSlash(srcPath)) // Normalize the paths used in the API.
	  
	  	resp, err := cli.get(ctx, "/containers/"+containerID+"/archive", query, nil)
	  	if err != nil {
	  		return nil, container.PathStat{}, err
	  	}
	  
	  	// In order to get the copy behavior right, we need to know information
	  	// about both the source and the destination. The response headers include
	  	// stat info about the source that we can use in deciding exactly how to
	  	// copy it locally. Along with the stat info about the local destination,
	  	// we have everything we need to handle the multiple possibilities there
	  	// can be when copying a file/dir from one location to another file/dir.
	  	stat, err := getContainerPathStatFromHeader(resp.Header)
	  	if err != nil {
	  		return nil, stat, fmt.Errorf("unable to get resource stat from response: %s", err)
	  	}
	  	return resp.Body, stat, err
	  }
	  ```
- **参数：**
	- `srcPath string`：容器内源文件或目录的路径。
- **返回值：**
	- `io.ReadCloser`：TAR 归档流。包含容器内路径内容的读取器。调用者必须负责关闭此读取器以释放资源。
	- `container.PathStat`：源路径的文件状态信息。
- **注意：**
	- **返回格式：**无论是复制单个文件还是一个目录，Docker API 返回的内容始终是一个 TAR（Tape Archive）归档流。调用者需要使用 Go 的 `archive/tar` 包来处理和解压这个流。
	- **元数据作用：**返回的 `container.PathStat` 元数据至关重要。它告诉主机程序源路径是文件还是目录，以及它的权限、时间戳等。
	- **API 端点：** 此函数和 `ContainerStatPath` 共享同一个 API 端点 `/archive`。区别在于：
		- `CopyFromContainer` 使用 **GET** 方法获取内容。
		- `ContainerStatPath` 使用 **HEAD** 方法仅获取元数据。