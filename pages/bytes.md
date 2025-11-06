- **有什么用？**
	- `bytes` 包是 Go 标准库中专门用来处理字节切片（`[]byte`）的工具包。
- `func NewReader(b []byte) *Reader`
  collapsed:: true
	- **作用：**
		- 把一个字节切片封装成一个可读的数据流（`Reader`），让它能像文件或网络连接一样被顺序读取，方便传给需要 `io.Reader` 接口的函数。
	- **参数：**
		- `b []byte`：要被读取的字节切片数据。
	- **返回值：**
		- `*Reader`：一个用来“读数据”的对象，它把一段字节切片（`[]byte`）包装成像文件一样可顺序读取的数据流。就像一个“光标”，能记住当前读到哪里，每次调用 `Read` 都从上次停下的地方继续往后读。
	- **注意：**
		-
	- **代码示例：**
		- ```go
		  package main
		  
		  import (
		  	"bytes"
		  	"fmt"
		  )
		  
		  func main() {
		  	data := []byte("Hello, Go!")
		  	reader := bytes.NewReader(data)
		  
		  	buf := make([]byte, 5)
		  	n, _ := reader.Read(buf)
		  	fmt.Printf("读到 %d 字节：%s\n", n, string(buf)) // 输出：读到 5 字节：Hello
		  
		  	// 再读一次
		  	n, _ = reader.Read(buf)
		  	fmt.Printf("继续读取：%s\n", string(buf[:n])) // 输出：继续读取：, Go!
		  }
		  
		  ```
-