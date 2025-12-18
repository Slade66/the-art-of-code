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
		- **注意：**
			- 要正确处理 `time.Time`，您需要将 `parseTime` 作为参数包含在内。
			- 要完全支持 UTF-8 编码，您需要将 `charset=utf8` 更改为 `charset=utf8mb4`。
- ## DB
	- `type DB struct`
	  id:: 694256c8-9635-447a-8333-c080774525b3
	  collapsed:: true
		- **作用：**
			- `*gorm.DB` 是此时此刻的数据库操作上下文。它既包含了 “我要对数据库做什么（SQL条件）”，也包含了 “刚才数据库告诉了我什么（Error/RowsAffected）”。
			- 它扮演了以下 3 个主要角色：
				- **SQL 构建器：**
					- 它支持链式调用。当你调用 `Where`, `Select`, `Limit` 等方法时，GORM 并不是立即执行 SQL，而是通过这个对象不断叠加条件，组装成最终的 SQL 语句。
					- ```go
					  // db 是一个 *gorm.DB 对象
					  // 这里的 Where, Order, Limit 都会返回一个新的 *gorm.DB 对象
					  query := db.Where("age > ?", 18).Order("created_at desc").Limit(10)
					  ```
				- **执行结果的容器：**
					- 当你执行了“终结方法”（如 `Create`, `Find`, `Save`, `Delete`）后，GORM 会执行 SQL，并将执行的结果状态（是否报错、影响了多少行）存回返回的 `*gorm.DB` 对象中。
					- ```go
					  // result 也是一个 *gorm.DB 对象
					  result := db.Create(&user)
					  
					  // 通过 result (即 *gorm.DB) 获取执行结果
					  if result.Error != nil {
					      // 处理错误
					  }
					  fmt.Println(result.RowsAffected) // 获取影响行数
					  ```
				- **事务管理器：**
					- ```go
					  tx := db.Begin() // 开启事务，返回一个代表该事务的 *gorm.DB 对象
					  
					  //在此之后，必须使用 tx 来操作数据库，才能保证在同一个事务中
					  tx.Create(...)
					  tx.Update(...)
					  
					  tx.Commit() // 或者 tx.Rollback()
					  ```
		- **结构体定义：**
			- ```go
			  type DB struct {
			  	*Config
			  	Error        error
			  	RowsAffected int64
			  	Statement    *Statement
			  	clone        int
			  }
			  ```
		- **重要字段解析：**
			- `Error`：操作产生的错误。
				- 务必检查此项。
				- 如果插入过程中发生任何错误（如连接断开、约束冲突、Hook报错），这里会有值。
			- `RowsAffected`：受影响的行数。
		- **注意：**
			- **`*gorm.DB` 是并发安全的：**
				- 因为它使用了“克隆”机制。当你调用 `db.Where(...)` 时，GORM 不会修改原始的 `db` 对象，而是创建一个新的 `*gorm.DB` 实例（副本），把条件加在这个副本上并返回。
				- 这意味着你可以定义一个全局的 `db` 对象，然后在不同的请求中安全地复用它，而不用担心一个请求的查询条件污染了另一个请求。
				- ```go
				  // 基础的 db 对象（通常在程序启动时初始化）
				  var globalDB *gorm.DB 
				  
				  func GetUser() {
				      // tx 是一个新的 *gorm.DB 实例，包含了 "name = 'jinzhu'" 条件
				      tx := globalDB.Where("name = ?", "jinzhu")
				      
				      // globalDB 依然是干净的，不包含 Where 条件
				      // tx 则是“携带了状态”的会话
				      
				      tx.First(&user) // 执行查询
				  }
				  ```
	- `func (db *DB) Count(count *int64) (tx *DB)`
	  collapsed:: true
		- **作用：**根据当前 `db` 对象上已经拼好的查询条件，执行一条 `SELECT COUNT(*) ...`，并把匹配行数写进你传入的那个 `int64` 变量中。
		- **参数：**
			- `count *int64`：你传进来的整数变量的指针，GORM 会把统计出来的行数写进这个变量。
		- **返回值：**`*DB`：返回新的 `DB` 实例，方便继续链式调用。
		- **核心实现逻辑：**
			- **获取实例副本**：`Count` 第一步会调用 `db.GetInstance()` 或类似机制。这意味着它会基于当前的 `db` 复制出一个新的、临时的 `*gorm.DB` 对象（我们称之为 `tx`）。
			  logseq.order-list-type:: number
			- **修改副本**：在 `tx` 上，GORM 会强制把 `SELECT` 语句修改为 `SELECT COUNT(*)`。
			  logseq.order-list-type:: number
			- **执行查询**：执行 `tx` 的 SQL。
			  logseq.order-list-type:: number
			- **丢弃副本**：函数结束，`tx` 被销毁。
			  logseq.order-list-type:: number
			- 原始的 `db` 对象在这个过程中保持了“清白”，它根本不知道刚才发生了一次 `COUNT` 查询。
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
	- `func (db *DB) Updates(values interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**更新一个或多个字段。
			- 它生成的 SQL 类似于：`UPDATE users SET name='new', age=18 WHERE id=1;`
			- 它不会覆盖所有的列，只覆盖你传给它的那些列。
		- **代码示例：**
			- **单行记录更新：**
				- 要更新单行数据，你通常需要先通过 `Model` 指定要操作的对象（主要是为了获取主键 ID）。
				- ```go
				  var user User
				  db.First(&user, 1) // 查出 ID 为 1 的用户
				  
				  // 更新 Name 和 Age
				  // SQL: UPDATE users SET name='hello', age=18, updated_at='2024-...' WHERE id = 1;
				  db.Model(&user).Updates(User{Name: "hello", Age: 18})
				  ```
			- **批量更新：**
				- 此时它不需要 `Model` 中包含 ID，只需要指定表（Model）和条件。
				- ```go
				  // SQL: UPDATE users SET role='active', age=18 WHERE age < 10;
				  // 将所有 age < 10 的用户，年龄设为 18，状态设为 active
				  db.Model(&User{}).Where("age < ?", 10).Updates(User{Role: "active", Age: 18})
				  ```
		- **注意：**
			- **结构体的“零值陷阱”：**
				- 当你传入一个 `Struct` 给 `Updates` 时，GORM 会自动忽略零值（0, false, ""）。GORM 认为你是想“保持原样”，而不是“更新为 0”。
				- ```go
				  // 你的意图：把库存设置为 0，把激活状态设为 false
				  // 实际结果：什么都没变！因为 0 和 false 被忽略了。
				  db.Model(&product).Updates(Product{Stock: 0, IsActive: false})
				  ```
				- 如果你确实要把某个字段更新为 `0` 或 `false`：
					- **使用 Map：**
						- `map[string]interface{}` 包含所有的键值对，GORM 不会忽略其中的零值。
						- ```go
						  // SQL: UPDATE products SET stock=0, is_active=false ...
						  db.Model(&product).Updates(map[string]interface{}{
						    "stock": 0, 
						    "is_active": false,
						  })
						  ```
					- **使用 Select 指明字段：**
						- 告诉 GORM：“不管这个字段值是什么，我都强制更新它”。
						- ```go
						  // 强制更新 Stock 和 IsActive，即使它们是 0 或 false
						  db.Model(&product).Select("Stock", "IsActive").Updates(Product{
						    Stock: 0, 
						    IsActive: false,
						  })
						  
						  // 或者使用 "*" 更新所有字段
						  db.Model(&product).Select("*").Updates(Product{...})
						  ```
			- **安全提示：**如果没有任何 `Where` 条件，GORM 默认会阻止全局更新（Global Update）以防删库跑路。如果真的要全表更新，需要加 `AllowGlobalUpdate` 模式或者 `Where("1 = 1")`。
			- **更新时触发的钩子：**
				- 调用 `Updates` 时，会依次触发以下钩子：
					- `BeforeSave`
					- `BeforeUpdate`
					- **DB UPDATE 操作**
					- `AfterUpdate`
					- `AfterSave`
	- `func (db *DB) WithContext(ctx context.Context) *DB`
	  collapsed:: true
		- **作用：**
			- 将上下文（Context）传递给数据库操作，以便实现超时控制、取消操作或链路追踪等功能。
			- **超时控制**：结合 `context.WithTimeout`，可以防止数据库查询执行时间过长。
			- **请求取消**：当 HTTP 请求被取消时（例如用户断开连接），可以利用 Context 级联取消正在进行的数据库操作。
			- **链路追踪**：在微服务架构中，通过 Context 传递 Trace ID，以便监控数据库调用的性能和路径。
		- **注意：**
			- `WithContext` 会返回一个新的 `*gorm.DB` 实例，该实例绑定了你传入的 `context.Context`。这使得该 Context 仅对后续链式调用的操作生效，而不会影响原始的 `*gorm.DB` 实例。
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
	- `func (db *DB) Create(value interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**
			- 用于向数据库插入记录。
		- **参数：**
			-
		- **返回值：**
			- `tx *DB`： ((694256c8-9635-447a-8333-c080774525b3))
		- **代码示例：**
			- **插入单条记录：**
				- ```go
				  type User struct {
				    ID           uint
				    Name         string
				    Email        string
				    Age          uint8
				    CreatedAt    time.Time
				  }
				  
				  func main() {
				    user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
				  
				    // 注意：必须传入指针 &user
				    result := db.Create(&user) 
				    
				    // 检查错误和返回结果
				    if result.Error != nil {
				        // 处理错误
				    }
				    fmt.Println(user.ID)             // 插入成功后，GORM 会自动回填主键 ID
				    fmt.Println(result.RowsAffected) // 返回插入的记录数
				  }
				  ```
			- **批量插入：**
				- GORM 允许你通过传入一个 Slice（切片）来一次性插入多条数据。GORM 会自动生成一条 SQL 语句来插入所有数据（例如 `INSERT INTO users (...) VALUES (...), (...), ...`），这比循环插入单条数据效率高得多。
				- ```go
				  users := []User{
				    {Name: "Jinzhu", Age: 18},
				    {Name: "Jackson", Age: 19},
				  }
				  
				  result := db.Create(&users) // 传入切片的指针
				  
				  fmt.Println(result.RowsAffected) // 输出 2
				  for _, user := range users {
				      fmt.Println(user.ID) // ID 也会被回填
				  }
				  ```
			- **只插入指定字段：**
				- ```go
				  // INSERT INTO users (name, age) VALUES ("jinzhu", 18)
				  db.Select("Name", "Age").Create(&user)
				  ```
			- **忽略指定字段插入：**
				- ```go
				  // INSERT INTO users (name, age) VALUES ("jinzhu", 18) 
				  // 忽略了 Email 和 CreatedAt
				  db.Omit("Email", "CreatedAt").Create(&user)
				  ```
		- **注意：**
			- **必须传入指针：**只有传入指针，GORM 才能在数据插入数据库后，将数据库生成的 `ID`（主键）和 `CreatedAt` 等默认值写回到原本的结构体中。
			- **触发钩子函数：**
				- GORM 在创建数据的过程中会触发一系列的钩子函数。这对于数据验证、自动生成 UUID 或密码加密非常有用。
				- 执行顺序如下：
					- `BeforeSave`
					- `BeforeCreate`
					- **DB INSERT 操作**
					- `AfterCreate`
					- `AfterSave`
				- 如果在任何 Hook 中返回错误，GORM 将停止插入操作并回滚事务。
				- ```go
				  func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
				    if u.Name == "admin" {
				      return errors.New("cannot create user with name admin")
				    }
				    u.UUID = uuid.New().String() // 自动生成 UUID
				    return
				  }
				  ```
	- `func (db *DB) Where(query interface{}, args ...interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**
			- `Where` 本质上只是往当前的 `*gorm.DB` 操作上下文里不断追加查询条件，像是在完善一份 SQL 草稿，直到真正执行查询或更新时，才把这些条件组合成最终的 SQL 发给数据库。
			- | **传入参数类型** | **示例** | **行为** | **备注** |
			  | ---- | ---- | ---- |
			  | **String** | `"name = ?", "tom"` | `WHERE name = 'tom'` | **推荐**，最清晰，无坑。 |
			  | **Struct** | `&User{Name: "tom"}` | `WHERE name = 'tom'` | **有坑**：GORM 只会查询非零值字段。`0`, `false`, `""` 这些在 Go 中被视为“零值”的字段，不会被构建到 SQL 中。 |
			  | **Map** | `{"age": 0}` | `WHERE age = 0` | 包含所有 Key，适合查询零值。 |
			  | **Slice** | `"id IN ?", []int{1,2}` | `WHERE id IN (1, 2)` | 自动展开为 IN 查询。 |
		- **注意：**
			- **内联 Where：**
				- 你不需要总是先写 `Where`。像 `First`, `Find`, `Last`, `Delete` 这样的“终结方法”都支持直接传入查询条件。
				- ```go
				  // 啰嗦的写法
				  db.Where("name = ?", "jinzhu").First(&user)
				  
				  // 大师写法 (内联)
				  db.First(&user, "name = ?", "jinzhu")
				  
				  // 根据主键查找 (最常用)
				  db.First(&user, 10) // id = 10
				  ```
	- `func (db *DB) Delete(value interface{}, conds ...interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**
			- `Delete` 方法用于根据主键或条件移除记录，若模型包含 `DeletedAt` 字段则默认执行软删除（仅更新时间戳隐藏数据），否则执行物理删除（从数据库彻底清除）。
		- **代码示例：**
			- **根据主键删除：**
				- 你需要告诉 GORM：哪张表以及删谁（ID）。
				- ```go
				  // 方法 A：内联主键 (最简洁)
				  // DELETE FROM users WHERE id = 10;
				  db.Delete(&User{}, 10)
				  
				  // 方法 B：传入包含 ID 的对象
				  user := User{ID: 10}
				  db.Delete(&user)
				  ```
			- **批量删除：**
				- 配合 `Where` 条件，可以一次性删除多条记录。
				- ```go
				  // DELETE FROM users WHERE name = 'jinzhu';
				  db.Where("name = ?", "jinzhu").Delete(&User{})
				  
				  // 或者使用切片传入多个 ID
				  // DELETE FROM users WHERE id IN (1,2,3);
				  db.Delete(&User{}, []int{1, 2, 3})
				  ```
		- **核心特性：软删除**
			- 如果你的模型结构体包含 `gorm.DeletedAt` 字段（或者嵌入了 `gorm.Model`），调用 `Delete` 时，GORM 不会真的从数据库删除这条记录！
			- 它会执行一个 `UPDATE` 操作，将 `deleted_at` 字段更新为当前时间。
			- ```go
			  // 嵌入了 gorm.Model，自带 Soft Delete 能力
			  type User struct {
			    gorm.Model 
			    Name string
			  }
			  
			  // 代码调用 Delete
			  db.Delete(&user)
			  
			  // 实际执行的 SQL 竟然是 Update！
			  // UPDATE users SET deleted_at="2024-10-29 10:00:00" WHERE id = 10;
			  ```
			- **软删除的效果：**
				- **普通查询看不到：**调用 `db.Find()` 时，GORM 会自动加上 `WHERE deleted_at IS NULL`，所以你查不到这些被“删掉”的数据。
				- **数据依然在：**你去数据库里直接写 SQL 查，数据还在那里，只是被打上了“已删除”的标记。
			- **找回软删除 & 物理删除**
				- **查询已删除的数据：**
					- ```go
					  // SELECT * FROM users WHERE id = 10; (不加 deleted_at IS NULL 条件)
					  db.Unscoped().First(&user, 10)
					  
					  // 查找所有数据（包含被软删除的）
					  db.Unscoped().Find(&users)
					  ```
				- **物理删除：**
					- ```go
					  // DELETE FROM users WHERE id = 10;
					  // 真的删了，找不回来了
					  db.Unscoped().Delete(&user)
					  ```
		- **注意：**
			- **安全机制：阻止全局删除**
				- 如果结构体的 `ID` 为空（零值），GORM 会触发批量删除（如果没有开启防全表删除保护，这会很危险）。务必确保主键有值。
				- 为了防止新手写出 `db.Delete(&User{})` 这种导致全表清空的恐怖代码，GORM 默认开启了保护机制。
				- 如果没有任何 `Where` 条件，也没有指定主键，GORM 会报错：`there is no missing where clause`。
	- `func (db *DB) Preload(query string, args ...interface{}) (tx *DB)`
	  collapsed:: true
		- **作用：**
			- 在查询主对象（比如用户）的时候，告诉 GORM：“顺便帮我去隔壁表把关联的数据（比如信用卡、订单、角色）也查出来，填到我的结构体字段里。”
		- **参数：**
			- `query string`：结构体中的字段名。
			- `args ...interface{}`：给关联查询加条件。
		- **返回值：**
			- `*gorm.DB`：返回当前数据库会话的指针。
				- 为了支持链式调用：你可以一直点下去 `db.Preload("A").Preload("B").Find(...)`。
		- **核心执行流程：**
			- **Has Many / Has One（一对多 / 一对一）**
			  collapsed:: true
				- **场景：**查询用户（User），同时查出他名下的信用卡（CreditCards）。
					- **User ID：**10、11
					- **关系：**一个用户 → 多张信用卡
				- **执行流程：**
					- **主查询：**
					  logseq.order-list-type:: number
						- **Go：**`db.Preload("CreditCards").Find(&users)`
						- **SQL：**`SELECT * FROM users;`
						- **动作：**GORM 先查询所有用户数据，并从结果中提取主键 ID：[10, 11]。
					- **子查询：**
					  logseq.order-list-type:: number
						- **SQL：**
							- ```sql
							  SELECT * FROM credit_cards
							  WHERE user_id IN (10, 11);
							  -- 核心：使用 IN 一次性查出所有相关信用卡
							  ```
						- **动作：**GORM 查询到所有属于这些用户的信用卡数据。
					- **内存组装：**
					  logseq.order-list-type:: number
						- **动作：**GORM 在内存中遍历这批信用卡记录。
						- 如果发现某张卡的 `user_id` 是 10，就把它放进 `users[0].CreditCards`。
						- 如果发现某张卡的 `user_id` 是 11，就把它放进 `users[1].CreditCards`。
			- collapsed:: true
			  
			  **Belongs To（多对一）**
				- **场景：**查询用户（User），同时查出他们所属的公司（Company）。
					- User 有 `CompanyID` 列
					- **关系：**多个用户 → 属于同一个公司
				- **执行流程：**
					- **主查询：**
					  logseq.order-list-type:: number
						- **Go：**`db.Preload("Company").Find(&users)`
						- **SQL：**`SELECT * FROM users;`
						- **动作：**GORM 遍历查询到的用户，提取他们的外键 CompanyID。
							- User A（CompanyID: 99）
							- User B（CompanyID: 99）
							- User C（CompanyID: 100）
							- 去重后的 Company ID 列表：[99, 100]
					- **子查询：**
					  logseq.order-list-type:: number
						- **SQL：**
							- ```sql
							  SELECT * FROM companies
							  WHERE id IN (99, 100);
							  -- 核心：只查询需要的公司，重复 ID 只查一次
							  ```
						- **动作：**GORM 查询到所有相关公司数据。
					- **内存组装：**
					  logseq.order-list-type:: number
						- **动作：**将查询到的公司对象分配给对应用户
						- 把查到的 `Company 99` 的指针赋给 User A 和 User B 的 `Company` 字段。
						- 把 `Company 100` 赋给 User C。
			- **Many to Many（多对多）**
			  collapsed:: true
				- **场景：**查询用户（User），同时查出他们掌握的语言（Languages）。
					- **表结构：**`users`、`languages`、`user_languages`（中间表）
				- **执行流程：**
					- **主查询：**
					  logseq.order-list-type:: number
						- **Go：**`db.Preload("Languages").Find(&users)`
						- **SQL：**`SELECT * FROM users;`
						- **动作：**提取用户主键 ID：[1, 2]
					- **子查询：**
					  logseq.order-list-type:: number
						- **动作：**这里 GORM 需要做一个 JOIN，因为它不知道哪个语言属于哪个用户，必须问中间表。
						- **SQL：**
							- ```sql
							  SELECT 
							  languages.*,              -- 查询语言详情
							  user_languages.user_id    -- 关键：标明属于哪个用户，否则内存中无法匹配
							  FROM languages
							  INNER JOIN user_languages 
							  ON user_languages.language_id = languages.id
							  WHERE user_languages.user_id IN (1, 2);
							  ```
					- **内存组装：**
					  logseq.order-list-type:: number
						- **动作：**GORM 遍历查询结果，将语言分配给对应用户。
						- 看到 `English` 且 `user_id=1` -> 塞进 User 1 的 `Languages`。
						- 看到 `Chinese` 且 `user_id=1` -> 塞进 User 1 的 `Languages`。
			- **嵌套 Preload**
			  collapsed:: true
				- **场景：**查询用户（User）及其订单（Orders），再查询订单中的商品（OrderItems）。
					- **Go：**`db.Preload("Orders.OrderItems").Find(&users)`
				- **执行流程：**
					- **递归查询：**
						- **Level 1：**
							- **SQL：**`SELECT * FROM users;`
							- **动作：**获取 User ID 列表
						- **Level 2：**
							- **SQL：**`SELECT * FROM orders WHERE user_id IN (...);`
							- **动作：**获取订单 ID 列表
						- **Level 3：**
							- **SQL：**`SELECT * FROM order_items WHERE order_id IN (...);`
							- **动作：**获取订单对应的商品数据
					- **内存组装：**
						- **动作：**先把 Items 塞到 Orders，再把 Orders 塞到 Users
		- **注意：**
			- **别想过滤主表**：
			  collapsed:: true
				- `Preload("Orders", "amount > 100")` 不会只查出有大额订单的用户。它会查出所有用户，只是那些没有大额订单的用户，其 `Orders` 字段为空。
			- **字段名大小写敏感**：
			  collapsed:: true
				- 不是数据库的列名，也不是表名，是 Go Struct 里的那个 Field Name。
				- 参数必须完全匹配 Struct 里的字段名（如 `"Orders"`），写成 `"orders"` 或数据库表名通常会报错。
- **为什么 DeletedAt 要加索引？**
  collapsed:: true
	- 因为你用了 **软删除（soft delete）**：数据并没真删，而是把 `deleted_at` 填上时间。
	- 之后你所有“正常查询”基本都会隐含一个条件：`WHERE deleted_at IS NULL`
	- 给 `deleted_at` 加索引的目的就是：让这类查询更快。
- **软删除**
  collapsed:: true
	- GORM 的软删除并不会执行 `DELETE` 语句，而是将记录的 `deleted_at` 字段更新为当前时间。
	- **查询时：**GORM 会自动加上 `WHERE deleted_at IS NULL`，让应用层“看”不到这条数据，从而认为数据已删除。
	- **物理上：**这条数据依然安然无恙地躺在数据库表中。
	- **软删除与唯一索引的冲突：**
		- 数据库的唯一索引（Unique Index）是物理约束。它的作用是确保某一列（或多列组合）的值在整个表中是唯一的，它只关心它管着的那一列，不关心你是否觉得这条数据“已删除”。
		- 当你对一行数据进行软删除时，这条记录实际上仍然存在于表中。此时如果再次创建一条在唯一索引列上取值相同的记录，应用层会认为这是一次新的插入操作，但在数据库看来，这条记录早已存在，只是被标记为“已删除”、对应用层不可见而已。结果就是违反了唯一性约束，数据库会抛出 `Duplicated Entry` 错误。
		- **解决方案：复合唯一索引**
			- **在 GORM 中的写法：**
				- ```go
				  type User struct {
				      ID        uint
				      // 将 name 和 deleted_at 设为同一个索引组
				      Name      string         `gorm:"uniqueIndex:idx_name_deleted_at"`
				      DeletedAt gorm.DeletedAt `gorm:"uniqueIndex:idx_name_deleted_at"`
				  }
				  ```
			- **原理**：将 `DeletedAt` 加入到唯一索引中，形成 `(Name, DeletedAt)` 的联合唯一索引。
			- **目标**：允许 `("Jerry", "2023-10-01")` 和 `("Jerry", NULL)` 共存。
			- **这种方案在 MySQL 的陷阱：**
				- 在 MySQL 中，唯一索引通常允许多个 NULL 值共存（因为 `NULL` 的含义是未知，所以 `NULL != NULL`）。
				- 如果 `DeletedAt` 是 `NULL`（代表活跃用户），MySQL 允许存在多行 `("Jerry", NULL)`。
				- 这意味着：你可以创建多个同名的活跃用户，这违背了唯一索引的初衷（我们只允许一个活跃的 Jerry）。
				- **修正：**
					- 你需要让 `DeletedAt` 字段不使用 `NULL`，而是使用默认值（例如 0）。
					- 将 `DeletedAt` 类型改为 `int64` (存储时间戳) 或其他非 NULL 类型。
					- 未删除时为 `0`，删除时为时间戳。
					- 这样 `("Jerry", 0)` 就变得严格唯一了。
- **分页查询缺少排序导致的数据重复和数据丢失问题**
  collapsed:: true
	- SQL 数据库（如 MySQL、PostgreSQL）不保证 `SELECT` 查询的默认返回顺序。
	- **问题现象：**当你请求第 1 页和第 2 页时，如果没有明确的 `ORDER BY`，数据库可能会根据磁盘读取顺序随机返回数据。这意味着：
		- **数据重复：**某条记录可能同时出现在第 1 页和第 2 页。
		- **数据丢失：**某条记录可能在翻页过程中被跳过，哪一页都没出现。
	- **修复方案：**在分页查询时，必须加上确定性的排序条件（通常是主键 ID 或创建时间）。
		- ```go
		  // 在 Find 之前加上 Order
		  // 推荐使用主键排序，或者 CreatedAt desc
		  if err := db.Order("id DESC").Offset(int(offset)).Limit(int(pageSize)).Find(&sites).Error; err != nil {
		      return nil, 0, err
		  }
		  ```
- **分页查询的 DoS 问题**
  collapsed:: true
	- 如果不限制 `pageSize` 上限，调用方传 `pageSize=100000`，会全表扫/大内存分配，影响服务稳定性。
	- **分页防护：**`pageSize` 设置最大值（比如 100 / 200）。
- **为什么 gorm.ErrRecordNotFound 透传到 HTTP 常常会变成 500？**
  collapsed:: true
	- `gorm.ErrRecordNotFound` 只是一个普通的 `error`，不携带 Kratos 需要的 `Reason/Code`，transport 不知道这是 404，通常按 “未知内部错误” 处理，也就是 HTTP 500。
	- 要避免 500，核心就是：在进入 transport 前，把业务错误统一转换成 Kratos errors。
- **最佳实践：**
  collapsed:: true
	- 更新之前不需要先查询记录是否存在，直接尝试更新，如果 `RowsAffected` 为 0，就说明不存在。
- [[Go 程序设计语言/GORM/多对一（Many-to-One）]]
- [[Go 程序设计语言/GORM/多对多（Many-to-Many）]]
- `type Tabler interface`
  collapsed:: true
	- ```go
	  type Tabler interface {
	  	TableName() string
	  }
	  ```
	- `TableName() string`
		- **作用：**
			- 用于给某个自定义模型（Struct）硬编码一个固定的数据库表名。
			- 如果你的模型结构体实现了 `Tabler` 接口（即定义了一个无参数的 `TableName` 方法），GORM 会在解析 Schema 时调用它来获取表名。
-