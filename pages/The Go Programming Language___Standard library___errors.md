- **`errors.New`：创建简单的静态字符串错误**
	- **源代码：**
		- ```go
		  // New returns an error that formats as the given text.
		  // Each call to New returns a distinct error value even if the text is identical.
		  func New(text string) error {
		  	return &errorString{text}
		  }
		  
		  // errorString is a trivial implementation of error.
		  type errorString struct {
		  	s string
		  }
		  
		  func (e *errorString) Error() string {
		  	return e.s
		  }
		  ```
	- 创建错误最基本的方式是使用 `errors` 包中的 `New` 函数。
	- 它接收一个静态字符串作为参数，并返回一个 `error` 类型的值。
	- `error` 实际上是一个包含字符串字段的结构体类型，并实现了 `Error()` 方法，用于返回其中存储的错误信息。
	- **注意：**
		- 即使传入的文本消息完全相同，每次调用 `errors.New` 返回的错误值都不一样。
			- ```go
			  err1 := errors.New("something went wrong")
			  err2 := errors.New("something went wrong")
			  // 这将打印 false，因为 errors.New 总是返回不同的值
			  fmt.Println(err1 == err2)
			  ```
	- **代码示例：**
		- ```go
		  package main
		  
		  import (
		  	"errors"
		  	"fmt"
		  )
		  
		  func main() {
		  	err := errors.New("出错了！")
		  	if err != nil {
		  		fmt.Print(err)
		  	}
		  }
		  ```
-
-