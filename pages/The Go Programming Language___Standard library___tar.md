- **有什么用？**
	- `tar` 包用于处理 tar 归档文件（tar archives）。
- **读写 tar 文件的代码示例：**
	- ```go
	  package main
	  
	  import (
	  	"archive/tar" // 导入 tar 包，用于创建和读取 tar 归档文件
	  	"bytes"        // 导入 bytes 包，用于创建内存缓冲区（bytes.Buffer）作为 tar 数据的存储介质
	  	"fmt"          // 导入 fmt 包，用于格式化输出
	  	"io"           // 导入 io 包，提供基本的 I/O 接口和操作，如 io.Copy 和 io.EOF
	  	"log"          // 导入 log 包，用于错误处理
	  	"os"           // 导入 os 包，用于访问标准输出 os.Stdout
	  )
	  
	  func main() {
	  	// --- 第一部分：创建和写入 tar 归档文件 ---
	  
	  	// 创建一个 bytes.Buffer 作为 tar 归档数据的内存存储。
	  	// 归档数据将首先写入这个缓冲区。
	  	var buf bytes.Buffer
	  	
	  	// 使用 tar.NewWriter 创建一个 tar.Writer，它将把 tar 格式的数据写入到 buf 中。
	  	tw := tar.NewWriter(&buf)
	  	
	  	// 定义要添加到归档中的文件列表及其内容。
	  	var files = []struct {
	  		Name, Body string
	  	}{
	  		{"readme.txt", "This archive contains some text files."},
	  		{"gopher.txt", "Gopher names:\nGeorge\nGeoffrey\nGonzo"},
	  		{"todo.txt", "Get animal handling license."},
	  	}
	  	
	  	// 遍历文件列表，将每个文件添加到 tar 归档中。
	  	for _, file := range files {
	  		// 1. 创建 tar.Header 结构体，包含文件的元数据。
	  		hdr := &tar.Header{
	  			Name: file.Name,
	  			// 文件权限：0600 表示文件所有者可读写，其他人无权限。
	  			Mode: 0600,
	  			// 文件大小：必须准确设置，告知 Reader 有多少字节数据跟随在头部之后。
	  			Size: int64(len(file.Body)),
	  		}
	  		
	  		// 2. 写入文件头部（Header）。
	  		if err := tw.WriteHeader(hdr); err != nil {
	  			log.Fatal(err)
	  		}
	  		
	  		// 3. 写入文件内容（Body）。
	  		// tw.Write 接收的是文件的实际数据。
	  		if _, err := tw.Write([]byte(file.Body)); err != nil {
	  			log.Fatal(err)
	  		}
	  	}
	  	
	  	// 4. 关闭 tar.Writer。
	  	// 这一步非常重要，它会写入归档结束标记（两个零块），并确保所有缓冲的数据都被刷新到 buf 中。
	  	if err := tw.Close(); err != nil {
	  		log.Fatal(err)
	  	}
	  
	  	// --- 第二部分：读取和迭代 tar 归档文件 ---
	  
	  	// 使用 tar.NewReader 创建一个 tar.Reader，它将从已填充的 buf 中读取 tar 归档数据。
	  	tr := tar.NewReader(&buf)
	  	
	  	// 开始循环迭代归档中的文件条目。
	  	for {
	  		// 1. 调用 Next() 移动到归档中的下一个文件条目，并返回其 Header。
	  		hdr, err := tr.Next()
	  		
	  		// 检查是否到达归档文件的末尾。
	  		if err == io.EOF {
	  			break // 归档结束，退出循环
	  		}
	  		
	  		// 检查 Next() 过程中是否发生其他错误（如文件头解析错误）。
	  		if err != nil {
	  			log.Fatal(err)
	  		}
	  		
	  		// 2. 打印当前文件条目的名称。
	  		fmt.Printf("Contents of %s:\n", hdr.Name)
	  		
	  		// 3. 读取当前文件条目的数据。
	  		// tar.Reader (tr) 在调用 Next() 后就成为了当前文件内容的 io.Reader。
	  		// io.Copy 会将 tr 中剩余的所有数据（即当前文件内容）复制到 os.Stdout。
	  		if _, err := io.Copy(os.Stdout, tr); err != nil {
	  			log.Fatal(err)
	  		}
	  		
	  		// 打印一个空行，分隔不同文件的输出。
	  		fmt.Println()
	  		
	  		// 注意： io.Copy(os.Stdout, tr) 成功完成后，tr 会自动读取完当前文件的所有数据，
	  		// 并丢弃任何填充字节，为下一次 tr.Next() 调用做好准备。
	  	}
	  
	  }
	  ```
-
-