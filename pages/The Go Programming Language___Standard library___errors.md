## `errors.New`：创建简单的静态字符串错误
	- 创建错误最基本的方式是使用 `errors` 包中的 `New` 函数 。它接收一个静态的字符串消息，并返回一个 `error` 类型的值。
	- **即使传入的文本消息完全相同，每次调用 `errors.New` 返回的错误值都不一样：**
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