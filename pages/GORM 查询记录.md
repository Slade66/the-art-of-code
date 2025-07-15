- **检索单个对象：**
	- 当你只想从数据库里获取一条记录时，可以使用 `First`、`Take` 或 `Last`。它们都会自动在生成的 SQL 查询后加上 `LIMIT 1`。
	- `db.First(&user)`：
		- 获取按主键升序排列的第一条记录。
		- SQL: `SELECT * FROM users ORDER BY id LIMIT 1;`
	- `db.Take(&user)`：
		- 获取一条记录，但不保证顺序。
		- SQL: `SELECT * FROM users LIMIT 1;`
		- 当你只关心是否有数据，而不关心是哪一条时，最方便使用这个方法。
	- `db.Last(&user)`：
		- 获取按主键降序排列的第一条记录（即表中的最后一条）。
		- SQL: `SELECT * FROM users ORDER BY id DESC LIMIT 1;`
	- **处理查询结果和错误：**
		- 如果 `First`、`Take`、`Last` 没有找到任何记录，GORM 会返回一个特定的错误：`gorm.ErrRecordNotFound`。
		- ```go
		  var user User
		  result := db.First(&user) // 尝试获取第一条记录
		  
		  // result.RowsAffected // 返回找到的记录数，这里是 0 或 1
		  // result.Error        // 如果没找到，这里就会包含 gorm.ErrRecordNotFound
		  
		  // 推荐用 errors.Is 来检查是否是“记录未找到”的错误
		  if errors.Is(result.Error, gorm.ErrRecordNotFound) {
		      fmt.Println("记录未找到！")
		  }
		  ```
		- 如果你不希望“未找到”被当成一个错误来处理，你可以用 `Find` 来代替：`db.Limit(1).Find(&user)`。如果没找到记录，`Find` 不会返回错误，`user` 对象会保持其零值状态。
- **按主键检索：**
	- **对于数字类型主键：**
		- ```go
		  // 获取主键为 10 的用户
		  db.First(&user, 10)
		  // SELECT * FROM users WHERE id = 10;
		  
		  // 获取主键在 [1, 2, 3] 范围内的所有用户
		  db.Find(&users, []int{1, 2, 3})
		  // SELECT * FROM users WHERE id IN (1, 2, 3);
		  ```
	- **如果主键已经存在于你的 struct 变量中：**
		- GORM 会自动提取它作为查询条件。
		- ```go
		  var user = User{ID: 10}
		  db.First(&user)
		  // SELECT * FROM users WHERE id = 10;
		  ```
	- **对于字符串类型主键：**
		- 为了防止 SQL 注入，应该使用 `?` 作为占位符。
		- ```go
		  db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
		  ```
- **检索全部/多条对象：**
	- 当你需要获取一个表的所有记录，或满足某个条件的多条记录时，使用 `Find` 方法，并将一个切片的指针作为参数传入。
	- ```go
	  var users []User
	  
	  // 获取 users 表中的所有记录
	  result := db.Find(&users)
	  // SELECT * FROM users;
	  
	  fmt.Printf("找到了 %d 条用户记录\n", result.RowsAffected)
	  ```
- **条件查询（`Where`）：**
	- **字符串条件：**
		- 使用 `?` 作为参数占位符可以有效防止 SQL 注入。
		- ```go
		  // 等于
		  db.Where("name = ?", "jinzhu").First(&user)
		  
		  // 不等于
		  db.Where("name <> ?", "jinzhu").Find(&users)
		  
		  // IN
		  db.Where("name IN ?", []string{"jinzhu", "jinzhu 2"}).Find(&users)
		  
		  // LIKE
		  db.Where("name LIKE ?", "%jin%").Find(&users)
		  
		  // AND
		  db.Where("name = ? AND age >= ?", "jinzhu", 22).Find(&users)
		  
		  // BETWEEN
		  db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
		  ```
	- **Struct 和 Map 条件：**
		- 你也可以直接把 `struct` 或 `map` 作为查询条件。
			- ```go
			  // Struct 作为条件
			  db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
			  // SELECT * FROM users WHERE name = "jinzhu" AND age = 20;
			  
			  // Map 作为条件
			  db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
			  // SELECT * FROM users WHERE name = "jinzhu" AND age = 20;
			  ```
		- 当使用 `struct` 作为查询条件时，GORM 只会使用那些非零值的字段。例如，如果 `Age` 是 `0`，`Name` 是 `""`，`Active` 是 `false`，这些字段将被忽略！
			- ```go
			  // Age: 0 是 int 的零值，所以这个条件会被 GORM 忽略
			  db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
			  // 生成的 SQL: SELECT * FROM users WHERE name = "jinzhu"; (age=0 的条件丢失了！)
			  ```
		- 如果你需要查询零值，必须使用 `Map` 或者字符串条件。
			- ```go
			  db.Where(map[string]interface{}{"Name": "jinzhu", "Age": 0}).Find(&users)
			  // 正确生成 SQL: SELECT * FROM users WHERE name = "jinzhu" AND age = 0;
			  ```
	- **`Not` 和 `Or` 条件：**
		- ```go
		  db.Not("name = ?", "jinzhu").Find(&users)
		  // SELECT * FROM users WHERE NOT name = "jinzhu";
		  
		  db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
		  // SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';
		  ```
- **链式方法：**
	- GORM 的强大之处在于你可以将各种方法像链条一样串联起来，构建出复杂的查询。
	- `Select`：
		- 默认情况下，GORM 会查询所有字段 (`SELECT *`)。为了提高性能，你应该只选择你需要的字段。
		- ```go
		  db.Select("name", "age").Find(&users)
		  // SELECT name, age FROM users;
		  ```
	- `Order`：
		- 指定查询结果的排序方式。
		- ```go
		  db.Order("age desc, name asc").Find(&users)
		  // SELECT * FROM users ORDER BY age desc, name asc;
		  ```
	- `Limit` 和 `Offset`：
		- `Limit(n)`：最多获取 `n` 条记录。
		- `Offset(n)`：跳过前 `n` 条记录。
		- ```go
		  // 获取第 2 页，每页 10 条记录
		  // (页码-1) * 每页数量 = (2-1) * 10 = 10
		  db.Limit(10).Offset(10).Find(&users)
		  // SELECT * FROM users LIMIT 10 OFFSET 10;
		  ```
	- `Group` 和 `Having`：
		- `Group(column)`：按指定列分组。
		- `Having(condition)`：对分组后的结果进行过滤。
		- ```go
		  type Result struct {
		    Date  time.Time
		    Total int
		  }
		  var results []Result
		  
		  db.Model(&Order{}).Select("date(created_at) as date, sum(amount) as total").
		      Group("date(created_at)").
		      Having("sum(amount) > ?", 100).
		      Scan(&results)
		  // SELECT date(created_at) as date, sum(amount) as total FROM `orders` GROUP BY date(created_at) HAVING sum(amount) > 100;
		  ```
	- `Joins`：
		- `Joins` 是 GORM 提供的一个查询构建方法，用于在生成的 SQL 语句中添加 `JOIN`（连接）子句。
		- `JOIN` 是 SQL 的核心功能，允许你根据共享的列（通常是外键）将两个或多个表中的行组合起来。
		- **方式一：原生 SQL 字符串**
			- 直接写入你想要的 `JOIN` 子句。这种方式最直观，也最灵活，你可以写任何复杂的 JOIN 逻辑。
			- ```go
			  type Result struct {
			      Name  string
			      Email string
			  }
			  var results []Result
			  
			  db.Model(&User{}).
			     Select("users.name, emails.email"). // 明确指定列，避免列名冲突
			     Joins("LEFT JOIN emails ON emails.user_id = users.id").
			     Scan(&results)
			  
			  // 生成的 SQL 大致如下:
			  // SELECT users.name, emails.email
			  // FROM `users`
			  // LEFT JOIN emails ON emails.user_id = users.id;
			  ```
		- **方式二：GORM 关联名**
			- 如果你的模型之间已经定义了关联关系（例如 `has many`, `belongs to`），你可以直接使用关联字段的名称。GORM 会自动根据你的模型定义生成 `JOIN` 语句。
			- ```go
			  // 假设 User struct 中有定义:
			  // type User struct {
			  //   gorm.Model
			  //   Name string
			  //   Emails []Email // Has Many 关联
			  // }
			  
			  db.Joins("Emails").Find(&users)
			  
			  // GORM 会自动分析模型，生成类似下面的 SQL:
			  // SELECT `users`.*, `Emails`.*
			  // FROM `users`
			  // LEFT JOIN `emails` AS `Emails` ON `users`.`id` = `Emails`.`user_id`
			  ```
	- `Scan`：
		- `Scan` 是一个结果映射方法。它的作用是将数据库查询返回的行和列数据，填充（“扫描”）到一个你指定的 Go `struct` 变量中。
		- **为什么要用它？**
			- `Find` 方法非常方便，但它有一个前提：查询返回的列必须能完整地映射到你提供的模型 `struct`。
			- 当你使用 `Joins`、`Group` 或复杂的 `Select` 时，查询结果的“形状”往往是自定义的，它不再对应任何一个单一的模型。
			- 例如，`SELECT users.name, emails.email` 的结果，既不是一个完整的 `User`，也不是一个完整的 `Email`。这时 `db.Find(&User{})` 或 `db.Find(&Email{})` 都无法正确接收数据。
			- `Scan` 就是为了解决这个问题而生的。它允许你定义一个临时的、专门用于接收这种自定义结果的 `struct`。
		- **怎么用？**
			- **步骤：**
				- 定义结果 Struct：根据你的 `SELECT` 语句，定义一个能承载结果的 `struct`。
				  logseq.order-list-type:: number
				- 构建查询：使用 GORM 构建你的查询，包括 `Select`, `Joins` 等。
				  logseq.order-list-type:: number
				- 调用 `Scan`：将定义好的结果 `struct` 的指针传递给 `Scan`。
				  logseq.order-list-type:: number
			- **代码示例：**
				- ```go
				  // 1. 定义一个专门用于接收结果的 struct
				  type UserEmailResult struct {
				      Name  string // 对应 SELECT users.name
				      Email string // 对应 SELECT emails.email
				  }
				  
				  var result UserEmailResult // 如果只查一条
				  // var results []UserEmailResult // 如果查多条
				  
				  // 2. 构建查询，并用 Scan 接收
				  db.Table("users"). // 使用 Table 或 Model 指定主表
				     Select("users.name, emails.email").
				     Joins("LEFT JOIN emails ON emails.user_id = users.id").
				     Where("users.name = ?", "jinzhu").
				     Scan(&result) // 3. 将结果扫描到 result 变量中
				  ```
	- `Model`：
		- `Model()` 是一个指定操作模型的方法。当你调用 `db.Model(&User{})` 时，实际上是在告诉 GORM：“接下来的所有操作（直到调用终结方法）都将围绕 User 这个模型进行。”
		- GORM 会从你传入的 `&User{}` 中提取所有元数据，为后续操作提供上下文。
		- **一旦指定了模型，GORM 就会为你启用一系列高级功能：**
		  id:: 687674d4-d982-444d-8720-53decca005c9
			- **自动推断表名：**GORM 会根据 `User` 结构体的名称，自动将其转换为蛇形复数形式的表名 `users`。你不需要手动写死表名。
			- **Hooks 调用：**如果 `User` 模型定义了钩子函数（如 `BeforeUpdate`, `AfterDelete`），在使用 `Model` 进行 `Update`, `Delete` 等操作时，这些钩子会被自动触发。
			- **软删除支持：**如果 `User` 模型包含了 `gorm.DeletedAt` 字段，那么 `db.Model(&User{}).Delete()` 操作会自动变成一次 `UPDATE ... SET deleted_at = ?`，而不是真正的物理删除。
			- **关联操作：**你可以使用关联名来进行 `Joins` 或 `Preload`，例如 `db.Model(&User{}).Joins("Emails")`，这样可以简化 `Joins` 语句的书写。
			- **字段上下文：**GORM 了解 `User` 结构体的所有字段，因此在执行 `Where`、`Select`、`Update` 等操作时，你可以直接使用结构体中的字段名，GORM 会自动将它们映射到数据库表中的相应列名。
	- `Table`：
		- `Table()` 是一个直接指定表名的方法。你给它一个字符串，它就在生成的 SQL 中使用这个字符串作为表名。
		- **它不会：**
			- 触发任何模型的 Hooks。
			- 执行软删除逻辑（`Delete` 就是物理删除）。
		- **什么时候用：**
			- 当你要查询的数据库表或视图，在你的 Go 项目中没有对应的 `struct` 模型时，`Table` 是唯一的选择。
			- 如果表名是在运行时动态生成或从配置中读取的，你只能用 `Table`。
			- 绕过 `Model` 的软删除逻辑。
-