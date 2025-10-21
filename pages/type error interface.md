- **源码：**
	- ```go
	  // The error built-in interface type is the conventional interface for
	  // representing an error condition, with the nil value representing no error.
	  type error interface {
	  	Error() string
	  }
	  ```
- **作用：**
	- `Error()` 方法的作用就是：当发生了错误，就调用这个方法来获取失败的原因，它会返回一个字符串，用文字清楚地描述“到底出了什么错”。
- **示例：**
	- ```go
	  package main
	  
	  import "fmt"
	  
	  // --- 1. 定义你的自定义错误结构体 ---
	  // 我们希望这个错误能携带 "哪个用户ID没找到" 的信息
	  type UserNotFoundError struct {
	  	UserID string
	  }
	  
	  // --- 2. [核心] 实现 error 接口 ---
	  // 你只需要为你的结构体（通常使用指针接收者）
	  // 添加一个 Error() string 方法。
	  func (e *UserNotFoundError) Error() string {
	  	// 返回你希望这个错误被打印时的样子
	  	return fmt.Sprintf("resource error: user with ID '%s' was not found", e.UserID)
	  }
	  
	  // --- 3. 编写一个模拟函数 ---
	  // 这个函数在找不到用户时，会返回你的自定义 error
	  func GetUserNameFromDB(id string) (string, error) {
	  	// 模拟数据库查询
	  	if id == "101" {
	  		return "Alice", nil // 找到了，error 返回 nil
	  	}
	  
	  	// 没找到！
	  	// 返回你自定义的错误结构体的实例
	  	return "", &UserNotFoundError{UserID: id}
	  }
	  
	  // --- 4. 在 main 函数中调用并检查 ---
	  func main() {
	  	// 模拟查询一个不存在的用户
	  	username, err := GetUserNameFromDB("404")
	  
	  	// 这是 Go 统一的错误检查方式
	  	if err != nil {
	  		// fmt.Println 会自动调用 err.Error()
	  		// 所以这里会打印出你在 Error() 方法中定义的字符串
	  		fmt.Println(err)
	  	} else {
	  		fmt.Println("User found:", username)
	  	}
	  }
	  ```
	- **定义：**创建一个 `struct`（如 `UserNotFoundError`）来存放需要的额外信息（例如 `UserID`）。
	- **实现：**为该 `struct` 添加 `Error() string` 方法，在其中使用 `fmt.Sprintf` 拼接清晰的错误信息。
	- **返回：**在函数（如 `GetUserNameFromDB`）中，当发生错误时，执行 `return "", &UserNotFoundError{...}`。
	- **检查：**调用方（如 `main` 函数）无需关心具体的错误类型，只需按照惯例检查 `if err != nil` 即可。
-