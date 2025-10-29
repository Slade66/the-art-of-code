- 预加载是为了解决“N+1 查询问题”，从而大幅提升程序性能和减少数据库压力。
- **N+1 查询问题：**
  collapsed:: true
	- 就是一次查询本来可以搞定的事情，却因为设计不当导致执行了大量不必要的数据库查询，浪费了大量的数据库资源，导致效率低下的情况。
	- **举例：**
		- 获取博客文章列表，并展示每篇文章下的所有评论。
		  id:: 6877b77c-a984-4d01-8ee3-a0fecc5be7a0
		- 这是一个典型的“一对多”关系：一篇文章拥有多条评论。
		- ```go
		  // Post 模型 (文章)
		  type Post struct {
		      gorm.Model
		      Title    string
		      Content  string
		      Comments []Comment // "一对多"关系：一篇文章拥有多条评论
		  }
		  
		  // Comment 模型 (评论)
		  type Comment struct {
		      gorm.Model
		      PostID  uint   // 外键，指向 Post 的 ID
		      Content string
		  }
		  
		  // ❌ 错误的实现 (导致 N+1 查询问题)
		  func demonstrateNPlusOne(db *gorm.DB) {
		  
		      // 1. 获取 3 篇文章 (这是“+1”查询)
		      var posts []Post
		      db.Limit(3).Find(&posts)
		      fmt.Printf("找到了 %d 篇文章。\n", len(posts))
		  
		      // 2. 遍历每一篇文章，单独为它查询评论 (这是“N”次查询)
		      fmt.Println("开始为每篇文章查询评论...")
		      for i := range posts {
		          // 每一次循环，都会触发一次新的数据库查询！
		          db.Where("post_id = ?", posts[i].ID).Find(&posts[i].Comments)
		      }
		  
		      fmt.Println("数据获取完成！")
		      fmt.Println("\n--- GORM 日志会显示类似如下的 SQL ---")
		      fmt.Println("SELECT * FROM `posts` LIMIT 3;") // 第1次查询
		      fmt.Println("SELECT * FROM `comments` WHERE `post_id` = 1;") // 第2次查询
		      fmt.Println("SELECT * FROM `comments` WHERE `post_id` = 2;") // 第3次查询
		      fmt.Println("SELECT * FROM `comments` WHERE `post_id` = 3;") // 第4次查询
		      fmt.Println("\n总查询次数: 1 (查文章) + 3 (查评论) = 4 次")
		  }
		  ```
		- **问题分析**：
			- **第 1 次查询 (`+1`)**：获取了 3 篇文章。
			- **接下来 N 次查询 (`N`)**：代码进入循环，有多少篇文章，就需要执行多少次额外的查询来获取评论。在这个例子中，N=3。
			- **总计**：`1 + 3 = 4` 次数据库查询。如果文章数量是 100，就会有 101 次查询，性能会非常糟糕。
- **使用 Preload 预加载：**
  collapsed:: true
	- `Preload` 是用来“预先加载”关联数据的，目的是一次性获取主数据和它所有相关联的数据。
	- 它告诉 GORM：“当你在查询主表（比如 `文章`）时，请顺便也把它们关联的从表（比如 `评论`）数据，用最少的查询次数一并取出来。”
	- 现在，我们使用 GORM 的 `Preload` 功能来重构上面的代码：
		- 通过 `IN` 子句批量查询所有评论信息。假设你已经得到了所有用户的 ID，可以通过一次查询获取所有评论。
		- ```go
		  func fixWithPreload(db *gorm.DB) {
		      fmt.Println("\n--- ✅ 正确的实现 (使用 Preload 预加载) ---")
		  
		      var posts []Post
		  
		      // 只用一行 .Preload("Comments") 就解决了问题！
		      db.Preload("Comments").Limit(3).Find(&posts)
		  
		      fmt.Println("数据获取完成！")
		      fmt.Println("\n--- GORM 日志会显示类似如下的 SQL ---")
		      fmt.Println("SELECT * FROM `posts` LIMIT 3;") // 第 1 次查询，获取 3 篇文章
		    	// 第 2 次查询，GORM 收集到所有文章的 ID [1, 2, ..., 10]，然后用一条 IN 语句，一次性把所有相关的评论全部取回。
		      fmt.Println("SELECT * FROM `comments` WHERE `post_id` IN (1, 2, 3);")
		      fmt.Println("\n总查询次数: 1 (查文章) + 1 (查所有相关评论) = 2 次")
		  }
		  ```
		- **优势分析**：
			- GORM 首先执行第 1 次查询获取所有文章。
			- 然后，它会收集所有文章的 ID，用一条 `IN` 语句，将所有相关的评论一次性全部查询出来。
			- **总计**：无论有多少篇文章，总的查询次数永远是 2 次（固定次数）！这极大地提升了效率。
	- 还可以使用 `JOIN` 查询，单次查询就可以同时获取所有文章及其评论的信息。
-