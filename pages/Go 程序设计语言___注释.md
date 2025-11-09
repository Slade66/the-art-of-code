- Go 的文档注释（`go doc` / `godoc`）有一套**非常严格但简单的标准**。掌握它能让你的包自动生成清晰的 API 文档（Go 官方网站、pkg.go.dev 都是靠这个生成的）。
- ## Go 的注释原则
	- **每个导出标识符（即首字母大写的变量、函数、类型、常量、结构体、接口等）都应该有注释。**
	- **注释必须紧贴在被注释对象的上一行，中间不能有空行。**
	- collapsed:: true
	  
	  **注释必须以被注释的名字开头。**
		- ```go
		  // Package calc provides basic arithmetic operations.
		  package calc
		  
		  // Add returns the sum of two integers.
		  func Add(a, b int) int {
		      return a + b
		  }
		  
		  // Sub subtracts b from a and returns the result.
		  func Sub(a, b int) int {
		      return a - b
		  }
		  
		  // Calculator represents a simple calculator with a name.
		  type Calculator struct {
		      Name string
		  }
		  
		  // Multiply returns the product of two integers.
		  func (c Calculator) Multiply(a, b int) int {
		      return a * b
		  }
		  
		  ```
	- **描述错误返回值。**
	  collapsed:: true
		- 如果函数返回错误，建议说明错误何时会发生。
		- ```go
		  // ParseConfig parses the given file into a Config struct.
		  // It returns an error if the file cannot be read or parsed.
		  func ParseConfig(filename string) (Config, error) { ... }
		  
		  ```
	- **说“做了什么”，不要写“怎么做的”（实现细节）。**
	- **多行注释的每一行都以 `//` 开头，中间不要留空行。**
	  collapsed:: true
		- ```go
		  // QueryUsers retrieves a list of users matching the given filters.
		  // Filters may include name, age range, or active status.
		  // It returns a slice of users and the total count of matched records.
		  // If the query fails, an error is returned.
		  func QueryUsers(ctx context.Context, filters QueryFilter) ([]User, int, error) {
		      ...
		  }
		  
		  ```
-