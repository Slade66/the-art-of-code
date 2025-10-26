- [[type Stringer interface]]
- `fmt.Errorf`
	- `fmt.Errorf` 函数允许使用格式化动词来构建错误字符串。
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