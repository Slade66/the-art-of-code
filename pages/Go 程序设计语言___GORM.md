- **概念：**
  collapsed:: true
	- GORM 是 Go 语言的 ORM（Object-Relational Mapping，对象关系映射）框架。
	- **ORM：**
		- ORM（Object-Relational Mapping，对象关系映射）允许你像操作普通对象一样操作数据库中的数据。类（结构体）对应数据库中的表，对象对应表中的一行，属性对应表中的字段。操作对象就相当于操作表中的数据，从而以面向对象的方式简化了数据库操作，易于理解和使用。
		- ORM 会将你的操作转化为 SQL 执行，避免了手动编写 SQL 语句的复杂性。
		- ORM 能屏蔽不同数据库之间的差异，我们只需面向 ORM 编程，而无需针对特定数据库编写 SQL，这使得在后期切换数据库时更加方便。
		- **ORM 的问题：**
			- 复杂的查询可能还是需要手动编写 SQL，而 ORM 生成的 SQL 可能不如手写的 SQL 高效。
			- 虽然 ORM 能让你几乎不再需要手写 SQL，但 SQL 仍然是必须懂的，只有这样你才能更好地使用 ORM。
- **安装：**
  collapsed:: true
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
  collapsed:: true
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
- ## DB
	- `func (db *DB) Count(count *int64) (tx *DB)`
	  collapsed:: true
		- **作用：**根据当前 `db` 对象上已经拼好的查询条件，执行一条 `SELECT COUNT(*) ...`，并把匹配行数写进你传入的那个 `int64` 变量中。
		- **参数：**
			- `count *int64`：你传进来的整数变量的指针，GORM 会把统计出来的行数写进这个变量。
		- **返回值：**`*DB`：返回新的 `DB` 实例，方便继续链式调用。
	- `func (db *DB) Offset(offset int) (tx *DB)`
	  collapsed:: true
		- **作用：**在当前查询上设置 SQL 的 `OFFSET` 子句，表示跳过前 `offset` 条记录再开始返回结果，常用于分页。
		- **参数：**
			- `offset int`：要跳过的记录数量。
		- **返回值：**`*DB`：返回新的 `DB` 实例，方便继续链式调用。
	- `func (db *DB) Limit(limit int) (tx *DB)`
	  collapsed:: true
		- **作用：**在当前查询上设置 SQL 的 `LIMIT` 子句，表示最多返回 `limit` 条记录，常用于控制每页条数。
		- **参数：**`limit int`：最多返回的记录数量。
		- **返回值：**`*DB`：返回新的 `DB` 实例，方便继续链式调用。
	- `func (db *DB) Find(dest interface{}, conds ...interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**根据当前 `db` 上已经拼好的查询条件（`Model`、`Where`、`Offset`、`Limit` 等），执行 `SELECT ... FROM ...` 查询，并把结果扫描到 `dest` 指向的变量里。可以查一行或多行。
		- **参数：**
			- `dest interface{}`：用来接收查询结果的目标变量指针。
			- `conds ...interface{}`：额外的查询条件，等价于在这次 Find 上再 `Where` 一次（一般你现在用链式 `Where` 比较多，这里可以不传）。
		- **返回值：**`*DB`：返回新的 `DB` 实例，方便继续链式调用。
	- `func (db *DB) Scopes(funcs ...func(*DB) *DB) (tx *DB)`
	  collapsed:: true
		-
	- `func (db *DB) SetupJoinTable(model interface{}, field string, joinTable interface{}) error`
	  collapsed:: true
		- 显式告诉 GORM：多对多的时候用哪个模型当 join table。
	- `func (db *DB) CreateInBatches(value interface{}, batchSize int) (tx *DB)`
	  collapsed:: true
		- GORM 会把大切片拆成若干批次，每一批执行一次 INSERT。
		- 如果有 300 条记录，`batchSize = 100`：
			- 第 1 批：插入 0~99
			- 第 2 批：插入 100~199
			- 第 3 批：插入 200~299
	- `func (db *DB) Update(column string, value interface{}) (tx *DB)`
	  collapsed:: true
		- 把单个列更新成指定值。
	- `func (db *DB) WithContext(ctx context.Context) *DB`
	  collapsed:: true
		- **作用：**
			- 将上下文（Context）传递给数据库操作，以便实现超时控制、取消操作或链路追踪等功能。
			- `WithContext` 会返回一个新的 `*gorm.DB` 实例，该实例绑定了你传入的 `context.Context`。这使得该 Context 仅对后续链式调用的操作生效，而不会影响原始的 `*gorm.DB` 实例。
			- **超时控制**：结合 `context.WithTimeout`，可以防止数据库查询执行时间过长。
			- **请求取消**：当 HTTP 请求被取消时（例如用户断开连接），可以利用 Context 级联取消正在进行的数据库操作。
			- **链路追踪**：在微服务架构中，通过 Context 传递 Trace ID，以便监控数据库调用的性能和路径。
		- **示例：**
			- ```go
			  // 创建一个 2 秒超时的 Context
			  ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
			  defer cancel()
			  
			  // 使用 WithContext 将 ctx 传递给查询
			  // 如果查询超过 2 秒未完成，操作将被取消并返回错误
			  var users []User
			  result := db.WithContext(ctx).Find(&users)
			  
			  if result.Error != nil {
			      // 处理错误，可能是 context.DeadlineExceeded
			  }
			  ```
	- `func (db *DB) Transaction(fc func(tx *DB) error, opts ...*sql.TxOptions) (err error)`
	  collapsed:: true
		- **作用：**将一系列数据库操作封装在一个闭包函数（closure）中，并自动处理事务的开启、提交（Commit）和回滚（Rollback）。
		- **逻辑：**
			- **开启事务**：调用底层 `Begin` 方法开启一个新的数据库事务。
			- **执行闭包**：执行用户传入的函数 `fc`。
			- **自动提交与回滚**：
				- **提交**：如果 `fc` 返回 `nil`（无错误），GORM 会自动提交事务 (`Commit`)。
				- **回滚**：如果 `fc` 返回任何 `error`，GORM 会自动回滚事务 (`Rollback`)。
				- **Panic 保护**：如果在执行 `fc` 期间发生 `panic`，GORM 会捕获该 panic 并强制回滚事务，防止连接泄漏或数据不一致。
		- **示例：**
			- ```go
			  db.Transaction(func(tx *gorm.DB) error {
			    // 注意：在事务块内使用 'tx' 而不是 'db'
			    if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
			      // 返回错误，事务将回滚
			      return err
			    }
			  
			    if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
			      return err
			    }
			  
			    // 返回 nil，事务将提交
			    return nil
			  })
			  ```
		-
	- `func (db *DB) First(dest interface{}, conds ...interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**获取按主键升序排列的第一条记录。
		- **逻辑**：
			- **自动排序**：它会自动添加 `ORDER BY <primary_key> ASC`（例如 `ORDER BY id`）。
			- **限制数量**：它会自动添加 `LIMIT 1`。
			- **内联条件**：如果提供了 `conds` 参数，它会将这些条件添加到 `WHERE` 子句中。
			- **错误处理**：如果没有找到记录，它**会**返回 `ErrRecordNotFound` 错误。这是它与 `Find` 方法（没找到不报错）的一个主要区别。
		- **示例：**
			- 获取用户表中的第一条记录（按主键排序）：
				- ```go
				  var user User
				  // SELECT * FROM users ORDER BY id LIMIT 1;
				  result := DB.First(&user)
				  
				  if result.Error != nil {
				      // 处理错误，例如 gorm.ErrRecordNotFound
				  }
				  ```
			- 您可以直接在 `First` 中传入查询条件，这相当于调用了 `Where`：
				- ```go
				  var user User
				  // SELECT * FROM users WHERE name = 'jinzhu' ORDER BY id LIMIT 1;
				  DB.First(&user, "name = ?", "jinzhu")
				  
				  // 或者使用主键作为内联条件
				  // SELECT * FROM users WHERE id = 10 ORDER BY id LIMIT 1;
				  DB.First(&user, 10)
				  ```
	- `func (db *DB) Model(value interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**
			- 告诉 GORM：“接下来的操作，是针对哪张表的？如果结构体中有 ID，就锁定那一行数据操作”
			- 自动生成 SQL 的 `WHERE` 主键条件：
				- 如果你传入的结构体**包含主键值**（例如 ID），`Model()` 会自动提取这个主键，并将其作为 SQL 的 `WHERE` 条件。
				- ```go
				  // 假设结构体是 User {ID, Name, Age}
				  user := User{ID: 10}
				  
				  // 代码：
				  db.Model(&user).Update("Name", "Tom")
				  
				  // 生成的 SQL：
				  // UPDATE users SET name = 'Tom' WHERE id = 10;
				  // (注意：因为它看到了 user.ID 是 10，所以自动加了 WHERE id = 10)
				  ```
				- 你不需要手动写 `db.Where("id = ?", user.ID).Update(...)`，GORM 帮你自动完成了。
			- 指定表名：
				- 如果你传入的结构体没有主键值（或者是空结构体 &User{}），Model() 的作用就仅仅是告诉 GORM “我要操作 users 这张表”。
				- ```go
				  // 代码：没有指定 ID，只是为了告诉 GORM 去操作 users 表
				  db.Model(&User{}).Where("age > ?", 18).Update("is_adult", true)
				  
				  // 生成的 SQL：
				  // UPDATE users SET is_adult = true WHERE age > 18;
				  ```
		- **Model vs Table**
			- `db.Table("users")` 也能指定表，为什么要用 `Model`？
			- | **特性** | **db.Model(&User{})** | **db.Table("users")** |
			  | ---- | ---- | ---- |
			  | **来源** | 基于 Go 结构体 | 基于字符串（表名） |
			  | **主键智能识别** | **支持** (如果结构体有 ID，自动加 WHERE) | 不支持 (必须手写 Where) |
			  | **Hooks (钩子)** | **触发** (如 `BeforeUpdate`) | 不触发 |
			  | **软删除** | **支持** (自动处理 `deleted_at`) | 不支持 (必须手动处理) |
			  | **类型安全** | 高 (重构结构体名时会自动匹配) | 低 (表名改了，字符串不会变，容易报错) |
		- **注意：**
			- **不更新结构体的零值字段**
				- `Model` 配合 `Updates` (结构体传参) 使用时，不会更新零值（0, "", false）。
				- ```go
				  user := User{ID: 1}
				  
				  // ❌ 错误做法：试图把 Age 改为 0
				  // GORM 会认为 0 是默认值，从而忽略在这个字段上的更新
				  db.Model(&user).Updates(User{Age: 0, Name: "Jerry"})
				  // SQL: UPDATE users SET name='Jerry' WHERE id = 1; (Age 没变！)
				  
				  // ✅ 正确做法 A：使用 Map（推荐）
				  db.Model(&user).Updates(map[string]interface{}{"Age": 0, "Name": "Jerry"})
				  
				  // ✅ 正确做法 B：使用 Select 强制选中
				  db.Model(&user).Select("Age").Updates(User{Age: 0})
				  ```
- ## Association
	- `func (db *DB) Association(column string) *Association`
	  collapsed:: true
		- **作用：**
			-
		- **示例：**
			-
	- `func (association *Association) Replace(values ...interface{}) error`
	  collapsed:: true
		- **作用：**
			-
		- **示例：**
			-
-