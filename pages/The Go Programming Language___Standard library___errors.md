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
- `errors.Is`：
	- **函数签名：**`func Is(err, target error) bool`
	- 此函数用于检查错误链中是否存在一个错误与 `target` 错误值相等。
	- `errors.Is` 会沿着 `err` 的 `Unwrap` 链进行遍历，如果链中的任何一个错误等于 `target`，则返回 `true` 。
	- **之前 (使用 `==`)：**
		- ```go
		  if err == io.ErrUnexpectedEOF {... } // 如果 err 被包装过，这里会失败
		  ```
	- **之后 (使用 `errors.Is`)：**
		- ```go
		  if errors.Is(err, io.ErrUnexpectedEOF) {... } // 即使 err 被包装，也能正确判断
		  ```
- `errors.As`：
	- **函数签名：**`func As(err error, target any) bool`
	- 此函数用于检查错误链中是否存在一个错误的类型可以赋值给 `target`。
	- 它是对类型断言 `if e, ok := err.(*MyError)` 的现代化替代方案。
	- `target` 必须是一个指向错误类型变量的指针。如果找到匹配的错误，`errors.As` 会将该错误的值赋给 `target` 指向的变量，并返回 `true`，从而允许开发者访问该错误类型的字段和方法。
	- **之前 (使用类型断言)：**
		- ```go
		  if e, ok := err.(*os.PathError); ok {... } // 如果 err 被包装过，这里会失败
		  ```
	- **之后 (使用 `errors.As`)：**
		- ```go
		  var pathErr *os.PathError
		  if errors.As(err, &pathErr) {
		      // 可以在这里使用 pathErr 的字段，例如 pathErr.Path
		     ...
		  }
		  ```
-