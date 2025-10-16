- **概念：**
	- GORM 是 Go 语言的 ORM（Object-Relational Mapping，对象关系映射）框架。
	- **ORM：**
		- ORM（Object-Relational Mapping，对象关系映射）允许你像操作普通对象一样操作数据库中的数据。类（结构体）对应数据库中的表，对象对应表中的一行，属性对应表中的字段。操作对象就相当于操作表中的数据，从而以面向对象的方式简化了数据库操作，易于理解和使用。
		- ORM 会将你的操作转化为 SQL 执行，避免了手动编写 SQL 语句的复杂性。
		- ORM 能屏蔽不同数据库之间的差异，我们只需面向 ORM 编程，而无需针对特定数据库编写 SQL，这使得在后期切换数据库时更加方便。
		- **ORM 的问题：**
			- 复杂的查询可能还是需要手动编写 SQL，而 ORM 生成的 SQL 可能不如手写的 SQL 高效。
			- 虽然 ORM 能让你几乎不再需要手写 SQL，但 SQL 仍然是必须懂的，只有这样你才能更好地使用 ORM。
- **安装：**
	- **安装 GORM 包：**
	  logseq.order-list-type:: number
		- ```bash
		  go get -u gorm.io/gorm
		  ```
	- **安装数据库驱动：**
	  logseq.order-list-type:: number
		- MySQL 驱动：
			- ```bash
			  go get -u gorm.io/driver/mysql
			  ```
- **连接到数据库：**
	- **MySQL：**
		- ```go
		  import (
		    "gorm.io/driver/mysql"
		    "gorm.io/gorm"
		  )
		  
		  func main() {
		    // dsn = Data Source Name
		    dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
		    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
		  }
		  ```
		- `func Open(dialector Dialector, opts ...Option) (db *DB, err error)`
			- `gorm.Open` 是 GORM 中用于打开数据库连接的函数。它的作用是创建一个数据库连接并返回一个 `*gorm.DB` 类型的对象（通常命名为 `db`），你可以通过这个对象执行数据库操作（例如：查询、插入、更新、删除等）。
			- **第一个参数**：`mysql.Open(dsn)`
				- `mysql.Open(dsn)` 是 GORM 提供的 MySQL 数据库驱动函数，它接受一个连接字符串（DSN，Data Source Name）并返回一个 `gorm.Dialector` 类型的对象，GORM 使用它来知道如何与 MySQL 数据库进行交互。
			- **第二个参数**：`&gorm.Config{}`
				- `&gorm.Config{}` 是一个指向 `gorm.Config` 结构体的指针，GORM 配置项通过这个结构体来设置。默认情况下，`gorm.Config{}` 是空的，即使用 GORM 的默认配置。
- [[GORM 声明模型]]
- [[GORM 创建记录]]
- [[GORM 查询记录]]
- [[GORM 更新记录]]
- [[GORM 删除记录]]
- [[GORM 高级查询]]
- [[GORM Belongs To 多对一]]
- [[GORM Has One 一对一]]
- [[GORM Has Many 一对多]]
- [[GORM Many To Many 多对多]]
- [[Preloading]]
- [[GORM 的事务]]
- [[GORM Context]]
- [[GORM 的迁移]]
- **外键放哪里？**
	- 外键放在从属的那一方。
	- 身份证属于公民，所以在身份证表中建立外键，而不是在公民表。
	- 多对多关系无法通过在两张主表里加外键来实现，必须引入第三张中间表。
-