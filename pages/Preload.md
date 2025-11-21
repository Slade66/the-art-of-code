## 有什么用？
	- 它可以让你在查询主表数据时，顺便把关联表的数据也一起查出来，填到结构体里。
	- 如果不使用 `Preload`，GORM 默认是“懒”的，它只会查你指定的那个表，关联的字段会保持为空。
- ## 例子：博客系统
	- ```go
	  package main
	  
	  import "gorm.io/gorm"
	  
	  // User: 作者
	  type User struct {
	      gorm.Model
	      Name  string
	      // 关联字段：这里是一个切片，代表一个人可以有多篇文章
	      Posts []Post `gorm:"foreignKey:UserID"` 
	  }
	  
	  // Post: 文章
	  type Post struct {
	      gorm.Model
	      Title  string
	      Content string
	      // 外键：这篇文章属于哪个 User ID
	      UserID uint 
	  }
	  ```
	- 假设数据库里现在有这样一条数据：
		- **用户：**老王 (ID: 1)
		- **文章 1：**《Go语言入门》 (属于老王)
		- **文章 2：**《GORM 教程》 (属于老王)
	- **如果不使用 Preload：**
		- GORM 默认是“懒惰”的。当你去查用户时，它只查用户表。
		- ```go
		  var user User
		  
		  // ❌ 错误示范：只查主表
		  db.First(&user, "name = ?", "老王")
		  
		  fmt.Printf("用户: %s\n", user.Name)
		  fmt.Printf("他写了 %d 篇文章\n", len(user.Posts))
		  ```
		- ```
		  用户: 老王
		  他写了 0 篇文章  <-- 😱 哎？文章呢？
		  ```
		- **为什么？**你只告诉数据库 `SELECT * FROM users`。GORM 看到 `Posts` 字段，但因为它在另一张表里，GORM 默认不会去动那张表，所以 `user.Posts` 是空的。
	- **使用 Preload：**
		- `Preload` 就是告诉 GORM：“帮我个忙，查这个人的时候，顺便把他的文章也查出来填进去。”
		- ```go
		  var user User
		  
		  // ✅ 正确示范：预加载 Posts
		  db.Preload("Posts").First(&user, "name = ?", "老王")
		  
		  fmt.Printf("用户: %s\n", user.Name)
		  fmt.Printf("他写了 %d 篇文章\n", len(user.Posts))
		  for _, post := range user.Posts {
		      fmt.Printf(" - 标题: %s\n", post.Title)
		  }
		  ```
		- ```
		  用户: 老王
		  他写了 2 篇文章
		   - 标题: 《Go语言入门》
		   - 标题: 《GORM 教程》
		  ```
		- **发生了什么？（SQL 层面）**
			- 当你执行那行代码时，GORM 在后台悄悄执行了 两条 SQL。
			- ```sql
			  # 第一步：查人
			  SELECT * FROM users WHERE name = '老王' LIMIT 1;
			  -- 假设查出来 ID 是 1
			  
			  # 第二步：查文章 (自动填入上一步的 ID)
			  SELECT * FROM posts WHERE user_id = 1;
			  ```
			- **第三步：组装**
				- GORM 把第二步查到的两篇文章，塞进了第一步那个 `user` 结构体的 `Posts` 字段里。
	- **进阶：如果还有一层呢？（嵌套 Preload）**
		- 假设每篇文章下面还有评论。
		- 模型变成了：用户 -> 文章 -> 评论。 你想查老王，顺便要他的文章，顺便还要文章里的评论。
		- ```go
		  // 使用点号 . 来深入下一层
		  db.Preload("Posts.Comments").First(&user, "name = ?", "老王")
		  ```
-