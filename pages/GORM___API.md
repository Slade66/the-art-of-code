## DB
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