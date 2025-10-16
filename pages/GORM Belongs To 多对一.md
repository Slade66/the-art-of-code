- **概念：**
	- `BELONGS TO` 是 `HAS ONE` 和 `HAS MANY` 的反向关系，表示一种“从属”关系。例如，“一个员工属于一个公司”或“一篇博客属于一个作者”。它建立了一个模型到另一个模型的从属连接。
	- 一个 `User` 拥有多个 `Email`（`HAS MANY`），而每个 `Email` 属于一个 `User`（`BELONGS TO`）。
	- 一个公司拥有多名员工（`Company HAS MANY User`），而每个员工属于一个公司（`User BELONGS TO Company`）。
	- `HAS ONE` / `HAS MANY` 与 `BELONGS TO` 的关键区别在于外键的位置：
		- 在 `HAS ONE` / `HAS MANY` 中，外键位于被拥有的模型。
		- 在 `BELONGS TO` 中，外键位于当前模型。
- **比喻：**
	- 想象你和你的身份证。
	- 你，作为一个“人”，可以有很多东西（多篇文章、多个订单）。
	- 但你的这张“身份证”，只属于你一个人。它上面印着你的信息。
	- “身份证”属于“你”。这就是 `Belongs To` 关系。在数据库里，“身份证”这张表里，会有一个字段叫 `person_id`，记录着你这个“人”的ID。
- **示例：**
	- 一个订单属于一个用户。
	- 要在 `Order` 中建立它属于 `User` 的关系，你需要做两件事：
		- **定义外键 `UserID`：**这是真正在数据库 `orders` 表里创建的一个列，用来存放用户 `ID`。GORM 默认会使用“父模型名” + “ID” (即 `UserID`)作为外键名。
		- **定义嵌套结构体 `User`：**这不会在数据库里创建列。它纯粹是给 GORM 自己看的，当你使用 `Preload` 预加载功能时，GORM 会把查到的 `User` 完整信息填充到这个字段里。
	- ```go
	  // “父”模型
	  type User struct {
	    gorm.Model
	    Name string
	  }
	  
	  // “子”模型，它属于 User
	  type Order struct {
	    gorm.Model
	    OrderNo string
	    Amount  float64
	    UserID  uint   // 1. 外键字段
	    User    User   // 2. 一个嵌套的 User 结构体
	  }
	  ```
- **重写外键：**
	- 当你的外键字段名不叫“模型名ID”这种标准名字时，用它来告诉 GORM 你的外键到底叫什么。
	- **示例：**
		- 我们的 `orders` 表里，表示用户的外键字段名叫 `customer_id`，而不是 `user_id`。
		- ```go
		  type Order struct {
		    gorm.Model
		    CustomerID uint   // 自定义的外键字段名
		    User       User   `gorm:"foreignKey:CustomerID"` // 用标签告诉 GORM
		  }
		  ```
		- `foreignKey:CustomerID` 这个标签，就是在跟 GORM 说：“别再傻傻地找 `UserID` 字段了，请使用 `CustomerID` 作为连接 `User` 表的外键。”
- **重写引用：**
	- 默认是拿“父表”的主键 `ID` 来关联，用这个可以改成拿父表的其他唯一字段（比如工号 `Code`）来关联。
	- **示例：**
		- `orders` 表不是通过 `users` 表的 `id` 来关联用户，而是通过 `users` 表里一个唯一的 `employee_code` (员工工号) 字段来关联。
		- ```go
		  type User struct {
		    gorm.Model
		    Name         string
		    EmployeeCode string `gorm:"unique"` // 员工工号，必须是唯一键
		  }
		  
		  type Order struct {
		    gorm.Model
		    UserEmployeeCode string // 这个字段里存的是员工工号，而不是用户ID
		    User             User   `gorm:"references:EmployeeCode;foreignKey:UserEmployeeCode"`
		  }
		  ```
		- `references:EmployeeCode` 告诉 GORM：“`Order` 表里的外键值，对应的不是 `User` 表的 `ID`，而是 `User` 表的 `EmployeeCode` 字段。”
		- `foreignKey:UserEmployeeCode` 则告诉 GORM，在 `Order` 结构体中，`UserEmployeeCode` 这个字段是用来存放那个外键值的。
- **外键约束：**
	- 在数据库层面建立强制规则，比如“如果删除了一个用户，他名下的所有订单该怎么办”。
	- **比喻：**
		- 你注销了你的手机号（删除用户）。运营商（数据库）需要一个明确的规定来处理你手机号下绑定的各种业务（关联的订单）。
		- **`ON DELETE: CASCADE`**：你注销手机号，所有绑定业务自动跟着全部注销。最“暴力”。
		- **`ON DELETE: SET NULL`**：你注销手机号，所有绑定业务记录里的“手机号”一栏被清空，变成“未知”。业务记录本身还在。
		- **`ON DELETE: RESTRICT`**：你想注销手机号，系统提示“对不起，您还有业务未解绑，不能注销！”。最“保守”。
	- **示例：**
		- 当一个 `User` 账户被删除时，他名下的所有 `Order` 记录的 `user_id` 应该被自动设置为 `NULL`。
		- ```go
		  type Order struct {
		    gorm.Model
		    OrderNo string
		    // 注意：要实现 SET NULL，外键必须是指针类型，这样才能接受 NULL 值
		    UserID  *uint
		    User    User `gorm:"constraint:OnUpdate:CASCADE,OnDelete:SET NULL;"`
		  }
		  ```
-