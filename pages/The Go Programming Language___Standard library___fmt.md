- [[type Stringer interface]]
- `fmt.Errorf`
	- `fmt.Errorf` 函数允许使用格式化动词来构建错误字符串。
	- **创建错误链：`%w` 动词与 `Unwrap` 方法**
		- **Go 1.13 之前的问题：**在 Go 1.13 之前，标准库在错误处理方面存在一个显著的短板：当使用 `fmt.Errorf("... %v", err)` 为一个错误添加上下文时，原始错误的类型信息会丢失，只剩下它的字符串表示 。这使得程序无法以编程方式检查错误的根本原因，开发者不得不退而求其次，依赖脆弱的字符串匹配。
		- Go 1.13 引入了错误包装（error wrapping）的官方支持，其核心是两个新元素：`%w` 格式化动词和 `Unwrap` 方法约定。
		- **`%w` 动词**：在 `fmt.Errorf` 函数中，使用 `%w` 动词可以创建一个包装了底层错误的新的错误值。`%w` 要求其对应的参数必须是一个 `error`。
		- **`Unwrap` 方法**：
			- 如果一个错误类型包含了另一个错误，它可以实现一个名为 `Unwrap` 的方法，该方法返回其内部包含的错误。如果 `e1.Unwrap()` 返回 `e2`，我们就说 `e1` 包装了 `e2`。
			- 通过 `%w` 创建的错误会自动实现 `Unwrap` 方法。这种机制允许将错误链接在一起，形成一个“错误链”（error chain），它就像一个单向链表，记录了错误从最初发生点到当前处理点的完整传播路径 。链上的每一层都添加了新的上下文，但并未破坏原始错误，从而为后续的检查和调试保留了全部信息。
		- 为了检查错误链，Go 1.13 在 `errors` 包中引入了两个新的、至关重要的函数：`errors.Is` 和 `errors.As`。它们会自动遍历整个错误链，使开发者不必手动解包。
	- **代码示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func findUser(id int) error {
		      if id <= 0 {
		          return fmt.Errorf("invalid user ID: %d", id)
		      }
		      //...
		      return nil
		  }
		  
		  func main() {
		  	err := findUser(0)
		  	fmt.Println(err)
		  }
		  
		  ```
-