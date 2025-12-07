- 枚举类型是一种具有固定数量可能值的类型，每个值都有一个唯一的名称。
- Go 语言本身没有独立的枚举类型，但可以使用现有的语言特性轻松实现枚举。
- **Go 的枚举是软枚举：**
	- Go 的枚举不是真正的“封闭集合”，运行时并没有底层机制能锁定取值范围，你可以把一个超出范围的整数强转赋值给它。
	- 因为 Go 倾向于简单的语言设计。它没有像 Java 或 Rust 那样复杂的枚举类型系统，而是选择信任程序员。
	- **如何防御？**
		- **防御性 Switch：**
			- 每当你处理枚举时，永远不要假设值一定是合法的。务必加上 `default` 分支。
			- ```go
			  func handleStatus(s Status) {
			      switch s {
			      case StatusPending:
			          // 处理等待
			      case StatusRunning:
			          // 处理运行
			      default:
			          // 🚨 捕获破坏者！
			          // 这里可以选择报错、panic、或者返回错误
			          fmt.Printf("错误：收到未知的状态值 %d\n", s)
			      }
			  }
			  ```
		- **添加验证方法：**
			- ```go
			  func (s Status) IsValid() bool {
			      switch s {
			      case StatusPending, StatusRunning: // 列出所有合法值
			          return true
			      default:
			          return false
			      }
			  }
			  
			  // 使用时
			  s := Status(999)
			  if !s.IsValid() {
			      // 直接拒绝执行后续逻辑
			      return fmt.Errorf("invalid status")
			  }
			  ```
- **创建和使用枚举：**
	- ```go
	  package main
	  
	  import "fmt"
	  
	  // Step 1: 定义类型
	  type Role int
	  
	  // Step 2: 定义枚举值
	  const (
	  	RoleGuest Role = iota // 0
	  	RoleUser              // 1
	  	RoleAdmin             // 2
	  )
	  
	  // Step 3: 实现 String() 方法 (让 fmt.Println 输出名字而不是数字)
	  func (r Role) String() string {
	  	// 定义名称映射表
	  	names := [...]string{ // 这是实现 String 接口最快的方法。因为 iota 产生的是 0, 1, 2，正好对应数组的索引下标。
	  		"Guest",
	  		"User",
	  		"Admin",
	  	}
	  
	  	// 防御性检查：防止强制转换的越界值 (例如 Role(99)) 导致 panic
	  	if r < 0 || int(r) >= len(names) {
	  		return fmt.Sprintf("Unknown(%d)", int(r))
	  	}
	  
	  	return names[r]
	  }
	  
	  // Step 4: 业务逻辑 (Switch 必须包含 default 以防止非法值)
	  func accessControl(r Role) {
	  	switch r {
	  	case RoleAdmin:
	  		fmt.Println("✅ 允许进入管理后台")
	  	case RoleUser:
	  		fmt.Println("✅ 允许进入用户中心")
	  	case RoleGuest:
	  		fmt.Println("⚠️ 请先登录")
	  	default:
	  		// 捕捉非法的枚举值
	  		fmt.Printf("❌ 错误：检测到非法身份 [%s]，拒绝访问\n", r)
	  	}
	  }
	  
	  func main() {
	  	// 场景 A: 正常使用
	  	myRole := RoleAdmin
	  	fmt.Printf("当前身份: %s (底层值: %d)\n", myRole, myRole) // 自动调用 String()
	  	accessControl(myRole)
	  
	  	fmt.Println("-------------------")
	  
	  	// 场景 B: 模拟非法入侵 (强制转换 int)
	  	hackerRole := Role(99)
	  	// String() 方法处理了越界，不会崩，显示 "Unknown(99)"
	  	fmt.Printf("伪造身份: %s\n", hackerRole)
	  	// Switch 的 default 分支拦截了逻辑
	  	accessControl(hackerRole)
	  }
	  ```
	- **`type X int`：**这是为了类型安全。虽然它底层是数字，但如果不强制转换，你不能直接把一个普通的 `int` 传给它，防止弄混。
	-
-