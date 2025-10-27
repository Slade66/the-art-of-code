- **作用：**
	- `CopyFromContainer` 的作用是将容器内指定路径（无论是单个文件还是整个目录）的内容一次性打包成 tar 流返回，用于将容器中的文件或目录结构完整复制到宿主机。
	- 由于它会返回整个目录树，因此目录越大，网络传输和内存消耗也会相应增加。
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
	- `container.PathStat`：表示源路径的元数据信息，实际来源于 TAR 归档流的第一个头部块，即由 `srcPath` 参数指定的文件或目录的头部数据。它用于描述源路径的类型（文件或目录）、权限、时间戳等属性。
- **注意：**
	- **返回格式：**无论是复制单个文件还是一个目录，Docker API 返回的内容始终是一个 TAR（Tape Archive）归档流。调用者需要使用 Go 的 `archive/tar` 包来处理和解压这个流。
	- **API 端点：** 此函数和 `ContainerStatPath` 共享同一个 API 端点 `/archive`。区别在于：
		- `CopyFromContainer` 使用 **GET** 方法获取内容。
		- `ContainerStatPath` 使用 **HEAD** 方法仅获取元数据。
	- **始终打包并顺序传输完整文件：**daemon 端会把容器内的路径打成一个 tar 流并“从头开始顺序写出”。服务器会把全量发给你，它不会只返回文件的一段，也没有 Range/offset 的能力。因此，你只能从头读到尾。