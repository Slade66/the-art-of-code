- `HAS MANY` 关系表示一个模型实体可以拥有多个其他模型实体的实例。
- **比喻：**一个用户可以拥有多篇文章，而每篇文章只属于一个用户，其中用户是“一”的那方，文章是“多”的那方。
- **如何定义 HAS MANY 模型：**
	- ```go
	  // User 模型 (代表 "一" 的一方)
	  type User struct {
	  	gorm.Model
	  	Name  string
	  	Posts []Post //  "Has Many" 关系！一个 User 拥有多篇 Post
	  }
	  
	  // Post 模型 (代表 "多" 的一方)
	  type Post struct {
	  	gorm.Model
	  	Title   string
	  	Content string
	  	UserID  uint // 外键字段，GORM 会自动用它来创建 User 和 Post 的关联
	  }
	  ```
	- `Posts []Post`：这是 "Has Many" 关系的核心。我们在 `User` 结构体中声明了一个 `Post` 的切片（Slice）。GORM 会根据这个字段推断出 `User` 和 `Post` 之间是一对多关系。
-