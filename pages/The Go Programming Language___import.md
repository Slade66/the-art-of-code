- 在 Go 中，你应该把所有需要导入的包（packages）放在一个 `import` 关键字后的圆括号 `()` 里，而不是每个包都写一个 `import`。
- ## 两种导入方式
	- #### 多个 `import` 语句
		- 这种方式为每一个需要导入的包都单独写一行 `import`。
		- ```go
		  package main
		  
		  import "fmt"
		  import "math"
		  
		  func main() {
		      fmt.Println("Hello")
		      fmt.Println(math.Pi)
		  }
		  ```
		- 这种写法是合法的，代码可以正常工作。但是，当你的项目依赖十几个包时，文件开头就会变成这样：
		- ```go
		  import "fmt"
		  import "math"
		  import "log"
		  import "os"
		  import "net/http"
		  // ... 还有很多行 import
		  ```
		- 看起来非常冗长和杂乱。
	- #### 分组导入语句
		- 这种方式，也叫“因式分解”式导入，是 Go 语言推荐的风格。它只使用一个 `import` 关键字，然后用一对圆括号 `()` 将所有要导入的包名包裹起来，每个包名占一行。
		- ```go
		  package main
		  
		  import (
		      "fmt"
		      "math"
		  )
		  
		  func main() {
		      fmt.Println("Hello")
		      fmt.Println(math.Pi)
		  }
		  ```
		- 你可以把 `import (...)` 看作一个代码块，这个块专门用来声明当前文件所依赖的所有外部包。
	- #### 为什么分组导入是“好风格”？
		- 虽然两种方式在功能上完全一样，但编程不仅仅是为了让机器读懂，更是为了让人读懂。分组导入之所以成为 Go 语言的惯例，主要是因为**简洁性**：它减少了 `import` 关键字的重复。只写一次 `import` 显然比写十几次要干净利落。
- `goimports`
	- `goimports` 这个官方工具会自动帮你格式化 `import` 语句，它遵循的规则是：
		- 将导入的包分为标准库（standard library）包和第三方（third-party）包。
		- 两组之间用一个空行隔开。
		- 每一组内部按字母顺序排序。
	- 这样做的好处是让代码的依赖结构更加清晰：一眼就能看出哪些功能是 Go 语言自带的，哪些是引入的外部框架或库。
	- **举例：**
		- ```go
		  package main
		  
		  import (
		      // 首先是标准库的包
		      "fmt"
		      "log"
		      "net/http"
		      "os"
		  
		      // 接着是第三方的包
		      "github.com/gin-gonic/gin"
		      "github.com/go-kratos/kratos/v2" // 比如你正在使用的 Kratos
		      "gorm.io/gorm"
		  )
		  ```
- 你不需要手动去维护这个格式。只要你配置好了你的开发环境（如 VS Code, GoLand），每次保存文件时，goimports 或 gofmt 工具会自动将你的 import 语句格式化成推荐的分组形式。所以，你只需要放心大胆地写代码，让工具来帮你处理这些风格问题就行了。
-