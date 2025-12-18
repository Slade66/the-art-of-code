- **核心原理：中间表（Join Table）**
  collapsed:: true
	- **多对多关系不能只在某一方简单地增加一个外键。**
		- 比如：一个用户可以会多种语言，一种语言也可能被多个用户使用。
		- 如果在用户表中只用一个字段存语言的 id，那就意味着一个用户只能对应一种语言；
		- 如果在语言表中用一个字段存多个用户的 id，又会变成在一个字段里塞一堆值，不仅不符合关系型数据库的设计规范，也不方便查询和维护。
		- 因此，多对多关系通常需要通过一张中间表来拆解，把“用户–语言”的关系单独存储，每一条记录只表示一个用户会一种语言，这样结构清晰、扩展性也更好。
	- 这时候，需要引入第三张表（中间表 / 连接表，Join Table）来维护这种多对多关系。
		- `users`：用于存储用户本身的数据。
		- `languages`：用于存储语言相关的数据。
		- `user_languages`：中间表，只负责记录用户和语言之间的对应关系，其中包含：
			- `user_id`
			- `language_id`
	- **定义模型：**
		- 你只需要在其中一方（或双方）定义 `many2many` 标签，GORM 就会自动帮你创建 “中间表”。
		- ```go
		  // User 用户
		  type User struct {
		      gorm.Model
		      Name      string
		      // 核心 Tag: many2many:表名
		      Languages []Language `gorm:"many2many:user_languages;"`
		  }
		  
		  // Language 语言
		  type Language struct {
		      gorm.Model
		      Name string
		      // 对方也可以定义反向关系（可选），方便反查
		      Users []*User `gorm:"many2many:user_languages;"`
		  }
		  ```
		- `many2many:user_languages` 里的名字就是数据库中自动创建的中间表表名。
		- **多对多关系的字段到底加在哪一边？**
			- **查询方向单一：**如果只需要从一边发起查询，比如只关心“用户会哪些语言”，可以只在用户这一侧定义多对多关系。
			- **查询方向双向：**如果两边都需要频繁查询，比如既要查“用户会哪些语言”，又要查“某种语言有哪些用户”，那就需要在两边都定义关系，但中间表的名称必须保持一致。
			- **数据库层面：**无论关系定义在一边还是两边，GORM 最终生成的都是同一张中间表，用来专门存储双方的对应关系。
- **关联操作**
  collapsed:: true
	- GORM 提供了一套专门的关联模式（Association Mode）来操作中间表（关系表）。
	- **代码示例：**
		- 假设我们已经有了用户 `user` (张三) 和两个语言 `langCN` (中文), `langEN` (英文)。
		- **添加关系：**
			- 张三学会了中文和英文。
			- ```go
			  // 开始操作 user 的 Languages 字段
			  err := db.Model(&user).Association("Languages").Append(&langCN, &langEN)
			  ```
			- **发生了什么？** GORM 会在 `user_languages` 表里插入两条记录：`(张三ID, 中文ID)` 和 `(张三ID, 英文ID)`。
			- 它不会重复创建 Language 数据，只会在中间表里连线。
		- **替换关系：**
			- 张三失忆了，现在只会英文，以前会的都忘了。
			- ```go
			  // 旧关系会被删除，新关系会被插入
			  err := db.Model(&user).Association("Languages").Replace(&langEN)
			  ```
			- **发生了什么？** GORM 会先在中间表删掉张三所有的记录，然后插入一条 `(张三ID, 英文ID)`。
		- **删除关系：**
			- 张三不再会说英文了（但“英文”这个语言本身还在数据库里，只是张三不会了）。
			- ```go
			  err := db.Model(&user).Association("Languages").Delete(&langEN)
			  ```
			- **发生了什么？** GORM 仅仅从中间表 `user_languages` 删除了 `(张三ID, 英文ID)` 这行记录。
		- **清空关系：**
			- 张三变成了哑巴，什么语言都不会了。
			- ```go
			  err := db.Model(&user).Association("Languages").Clear()
			  ```
			- **发生了什么？** 删除中间表中所有关于张三的记录。
		- **统计关系数量：**
			- 张三会说几种语言？
			- ```go
			  count := db.Model(&user).Association("Languages").Count()
			  ```
- **自定义中间表：**
  collapsed:: true
	- 有时候，默认的中间表结构满足不了需求。
		- 比如：用户关注了某个商品，中间表不仅要存 `user_id` 和 `product_id`，还要存 `created_at`（关注时间）。
		- 这时候你需要自定义结构体来作为中间表，并使用 `SetupJoinTable`。
	- 如果你的表不是通过 ID 关联，或者你想自定义中间表的列名。
		- ```go
		  type User struct {
		      gorm.Model
		      UserCode  string `gorm:"unique"` // 自己的唯一标识
		      
		      Languages []Language `gorm:"many2many:user_languages;foreignKey:UserCode;joinForeignKey:UserRef;references:Code;joinReferences:LangRef"`
		  }
		  
		  type Language struct {
		      gorm.Model
		      Code string `gorm:"unique"` // 对方的唯一标识
		      Name string
		  }
		  ```
		- **Tag 解释：**
			- **`many2many:user_languages`：**中间表叫 `user_languages`。
			- **`foreignKey:UserCode`：**我方（User）用哪个字段去关联？ $\rightarrow$ `UserCode`。
			- **`joinForeignKey:UserRef`：**在中间表里，映射我方数据的列名叫什么？ $\rightarrow$ 叫 `UserRef`。
			- **`references:Code`：**对方（Language）用哪个字段被关联？ $\rightarrow$ `Code`。
			- **`joinReferences:LangRef`：**在中间表里，映射对方数据的列名叫什么？ $\rightarrow$ 叫 `LangRef`。
		- 你需要写这么长的 Tag 来告诉 GORM 如何映射。如果用默认规约，一切都很简单；一旦打破规约，就要写很多配置。
- **查询**
	-
-