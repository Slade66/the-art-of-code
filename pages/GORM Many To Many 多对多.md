- 两个模型（或表）中的记录可以互相拥有多个对方的记录。
- **比喻（文章和标签）：**
	- 一篇文章可以有多个标签（例如，一篇关于 "Go 并发" 的文章可以有 "Go"、"并发"、"编程" 三个标签）；
	- 一个标签也可以应用于多篇文章（例如，"Go" 标签可以出现在 "Go 并发编程"、"GORM 入门" 等多篇文章上）。
- **数据库层面的实现：**
	- 如果在 `posts` 表里加一个 `tag_id` 字段，那么一篇文章只能对应一个标签。
	- 如果在 `tags` 表里加一个 `post_id` 字段，那么一个标签只能对应一篇文章。
	- 这两种情况都无法满足我们的需求。
	- 解决方案是引入第三张表，我们称之为“连接表”，这张连接表专门用来记录 `posts` 和 `tags` 之间的关系。
	- **posts 表（文章表）：**
		- | id (主键) | title |
		  | ---- | ---- |
		  | 1 | "GORM 入门" |
		  | 2 | "Go 并发编程" |
	- **tags 表（标签表）：**
		- | id (主键) | name |
		  | ---- | ---- |
		  | 10 | "Go" |
		  | 11 | "GORM" |
		  | 12 | "并发" |
	- **post_tags 表（连接表）：**
		- | post_id (外键, 指向 posts.id) | tag_id (外键, 指向 tags.id) |
		  | 1 | 10 |
		  | 1 | 11 |
		  | 2 | 10 |
		  | 2 | 12 |
	- **解读连接表：**
		- 第一行 `(1, 10)` 表示：ID 为 1 的文章（"GORM 入门"）拥有 ID 为 10 的标签（"Go"）。
		- 第二行 `(1, 11)` 表示：ID 为 1 的文章（"GORM 入门"）拥有 ID 为 11 的标签（"GORM"）。
		- 第三行 `(2, 10)` 表示：ID 为 2 的文章（"Go 并发编程"）拥有 ID 为 10 的标签（"Go"）。
- **GORM 中的实现：**
	- ```go
	  // Post 模型 (文章)
	  type Post struct {
	      gorm.Model
	      Title   string
	      Content string
	      // 关键: 使用 gorm 标签定义与 Tag 的多对多关系
	      // `many2many` 表示关系类型
	      // `post_tags` 是 GORM 将自动创建和管理的连接表名称
	      Tags    []Tag `gorm:"many2many:post_tags;"`
	  }
	  
	  // Tag 模型 (标签)
	  type Tag struct {
	      gorm.Model
	      Name  string
	      // 定义反向关系，方便从 Tag 查询到所有相关的 Post
	      Posts []Post `gorm:"many2many:post_tags;"`
	  }
	  ```
- **创建：**
	- ```go
	  // 准备一些标签
	  tagGo := Tag{Name: "Go"}
	  tagBackend := Tag{Name: "后端开发"}
	  tagGorm := Tag{Name: "GORM"}
	  
	  // 创建这些标签记录
	  db.Create(&[]Tag{tagGo, tagBackend, tagGorm})
	  
	  // 创建一篇文章，并直接关联上面创建的两个标签
	  post := Post{
	      Title:   "Go GORM 实践",
	      Content: "GORM 是一个非常强大的 Go ORM 库...",
	      Tags:    []Tag{tagGo, tagGorm}, // 直接在切片中放入要关联的对象
	  }
	  
	  // 当创建 post 时, GORM 会:
	  // 1. 在 'posts' 表中插入一条新纪录
	  // 2. 在 'post_tags' 连接表中插入两条记录 (post.ID, tagGo.ID) 和 (post.ID, tagGorm.ID)
	  db.Create(&post)
	  ```
-