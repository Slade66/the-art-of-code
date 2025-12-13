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
- **为什么 DeletedAt 要加索引？**
	- 因为你用了 **软删除（soft delete）**：数据并没真删，而是把 `deleted_at` 填上时间。
	- 之后你所有“正常查询”基本都会隐含一个条件：`WHERE deleted_at IS NULL`
	- 给 `deleted_at` 加索引的目的就是：让这类查询更快。
- ## 多对一
  collapsed:: true
	- 在 GORM（Go 的 ORM 库）中，“多对一”关系通常被称为 **Belongs To**（属于）。
	- **举例：**
		- 想象一下**员工（User）**和**部门（Department）**的关系：
			- 一个部门有**多**个员工。
			- 一个员工只**属于**（Belongs To）**一**个部门。
		- 从员工的角度看，这就是“多对一”关系。
	- 在 GORM 中实现 Belongs To 关系，关键在于**在“多”的一方（子表）中保存“一”的一方（父表）的主键作为外键**。
	- **外键字段：**
		- 通常命名为 [关联结构体名]ID
		- 这是数据库中实际存在的列（外键），存储公司的 ID。
	- **关联结构体字段：**
		- 用于存储查询到的 Company 对象，这是一个 GORM 的辅助字段。
		- 默认情况下它不会在数据库 users 表中生成列，但在查询时，GORM 会把对应的公司数据填充到这里。
	- **GORM 的默认约定：**
		- GORM 看到 `User` 结构体中有一个 `Company` 字段。
		- 它会自动寻找名为 `CompanyID` 的字段作为外键。
		- 它默认 `CompanyID` 对应 `Company` 表的 `ID` 字段。
	- **自定义外键：**
		- 如果你的字段名不一样，比如外键叫 `CompID`，你需要使用 Struct Tag 来告诉 GORM：
			- **`foreignKey`**: 指定当前结构体（User）中的哪个字段是外键。
			- **`references`**: 指定目标结构体（Company）中的哪个字段被引用（通常默认为 ID，一般不需要写）。
	- **GORM 关联字段使用指针或值的区别：**
		- | **特性** | **指针类型 (*Company)** | **值类型 (Company)** |
		  | ---- | ---- | ---- |
		  | **空值表现** | `nil` (真·空) | **零值** (ID为0, 字段为空串) |
		  | **JSON 输出** | `"Company": null` | `"Company": { "ID":0, ... }` |
		  | **代码安全性** | ⚠️ **低** (直接调用会 Panic，需判空) | ✅ **高** (永远不会崩，安全访问) |
		  | **语义表达** | "没有公司" | "有一个空的公司对象" |
		- 如果关联字段是指针，外键 ID 也用指针 (`CompanyID *int`)，这样数据库可以存 SQL `NULL`。
	- **代码示例：**
		- ```go
		  package main
		  
		  import (
		  	"encoding/json"
		  	"fmt"
		  	"gorm.io/driver/sqlite"
		  	"gorm.io/gorm"
		  )
		  
		  // ================= 模型定义 =================
		  
		  // 1. 公司表 (最基础)
		  type Company struct {
		  	ID   uint
		  	Name string
		  }
		  
		  // 2. 员工表 (中间层)
		  type User struct {
		  	ID   uint
		  	Name string
		  
		  	// --- 关联 A: 标准多对一 (User -> Company) ---
		  	// 使用指针 *int，允许为空 (NULL)
		  	CompanyID *int
		  	Company   *Company // 默认约定：关联 CompanyID -> Company.ID
		  	
		  	// --- 关键字段: 员工工号 ---
		  	// 这个字段将作为被 CreditCard 引用的目标
		  	UserCode string `gorm:"unique;not null"` 
		  }
		  
		  // 3. 工卡表 (测试 references)
		  type CreditCard struct {
		  	ID     uint
		  	Number string
		  
		  	// --- 关联 B: 特殊多对一 (CreditCard -> User) ---
		  	// 数据库里存的是 "U-1001" 这种工号，而不是 User 的 ID (1, 2, 3...)
		  	OwnerCode string 
		  
		  	// [重点配置]
		  	// foreignKey: 指明本结构体的 OwnerCode 是外键
		  	// references: 指明关联 User 结构体的 UserCode 字段 (而不是默认的 User.ID)
		  	Owner User `gorm:"foreignKey:OwnerCode;references:UserCode"`
		  }
		  
		  // ================= 主程序 =================
		  
		  func main() {
		  	// 初始化 SQLite 内存数据库
		  	db, _ := gorm.Open(sqlite.Open(":memory:"), &gorm.Config{})
		  	// 自动迁移表结构
		  	db.AutoMigrate(&Company{}, &User{}, &CreditCard{})
		  
		  	// ------------------------------------
		  	// 1. 准备数据
		  	// ------------------------------------
		  	
		  	// 创建公司
		  	google := Company{Name: "Google"}
		  	db.Create(&google)
		  
		  	// 创建员工 (关联公司 + 设置工号)
		  	// UserCode "GOOG-001" 是我们将要引用的关键
		  	alice := User{
		  		Name:      "Alice", 
		  		Company:   &google, 
		  		UserCode:  "GOOG-001", 
		  	}
		  	db.Create(&alice)
		  
		  	// 创建工卡 (关联员工)
		  	// 注意：我们直接赋值 OwnerCode 为 "GOOG-001"，完全没用到 alice.ID
		  	card := CreditCard{
		  		Number:    "CARD-9999",
		  		OwnerCode: "GOOG-001", 
		  	}
		  	db.Create(&card)
		  
		  	// ------------------------------------
		  	// 2. 查询验证
		  	// ------------------------------------
		  	fmt.Println("======= 开始查询 =======")
		  
		  	var resultCard CreditCard
		  	
		  	// 链式预加载：
		  	// 1. 加载 Owner (根据 CreditCard.OwnerCode -> User.UserCode)
		  	// 2. 加载 Owner.Company (根据 User.CompanyID -> Company.ID)
		  	db.Preload("Owner.Company").First(&resultCard, "number = ?", "CARD-9999")
		  
		  	// ------------------------------------
		  	// 3. 输出结果
		  	// ------------------------------------
		  	
		  	// 打印工卡信息
		  	fmt.Printf("工卡号: %s\n", resultCard.Number)
		  	fmt.Printf("关联工号(外键): %s\n", resultCard.OwnerCode)
		  	
		  	// 打印关联的员工信息
		  	fmt.Printf("持卡人姓名: %s\n", resultCard.Owner.Name)
		  	fmt.Printf("持卡人工号: %s\n", resultCard.Owner.UserCode)
		  	
		  	// 打印员工的公司信息
		  	if resultCard.Owner.Company != nil {
		  		fmt.Printf("所属公司: %s\n", resultCard.Owner.Company.Name)
		  	}
		  
		  	// 打印完整 JSON 结构以便观察
		  	printJSON(resultCard)
		  }
		  
		  func printJSON(v interface{}) {
		  	b, _ := json.MarshalIndent(v, "", "  ")
		  	fmt.Printf("\nJSON 结构:\n%s\n", string(b))
		  }
		  ```
- ## 多对多
  collapsed:: true
	- **核心原理：中间表（Join Table）**
	  collapsed:: true
		- 与“一对多”不同，多对多不能只在某一方增加一个外键。它必须引入第三张表——中间表。
		- 多对多必须有 `many2many` 标签，GORM 会自动管理中间表。
	- **代码示例：**
	  collapsed:: true
		- **用户（User）和语言（Language）之间是多对多关系：**一个用户可以使用多种语言（如英语、中文等），而一种语言也可以被多个用户使用。
		- ```go
		  package main
		  
		  import (
		  	"fmt"
		  	"gorm.io/driver/sqlite"
		  	"gorm.io/gorm"
		  )
		  
		  // Language (被引用的实体)
		  type Language struct {
		  	gorm.Model
		  	Name string
		  }
		  
		  // User (主实体)
		  type User struct {
		  	gorm.Model
		  	Name string
		  
		  	// [核心配置]
		  	// 1. 使用切片表示多个
		  	// 2. 必须加 `many2many:表名` 标签
		  	// GORM 会自动创建名为 `user_languages` 的中间表
		  	Languages []*Language `gorm:"many2many:user_languages;"`
		  }
		  
		  func main() {
		  	db, _ := gorm.Open(sqlite.Open(":memory:"), &gorm.Config{})
		  	// GORM 会自动创建三张表：users, languages, user_languages
		  	db.AutoMigrate(&User{}, &Language{})
		  
		  	// ==========================================
		  	// 1. 插入 (Create) - 建立关系
		  	// ==========================================
		  	// 准备两种语言
		  	cn := Language{Name: "Chinese"}
		  	en := Language{Name: "English"}
		  
		  	user := User{
		  		Name: "John",
		  		// 直接把语言对象放进去，GORM 会自动处理中间表
		  		Languages: []*Language{&cn, &en},
		  	}
		  	db.Create(&user)
		  	
		  	fmt.Println("创建完成，中间表已自动填充。")
		  
		  	// ==========================================
		  	// 2. 查询 (Read) - 预加载
		  	// ==========================================
		  	var u User
		  	// 使用 Preload 加载关联数据
		  	db.Preload("Languages").First(&u, "name = ?", "John")
		  
		  	fmt.Printf("用户: %s 会说的语言: ", u.Name)
		  	for _, l := range u.Languages {
		  		fmt.Printf("[%s] ", l.Name)
		  	}
		  	fmt.Println()
		  
		  	// ==========================================
		  	// 3. 更新关联 (Update Association)
		  	// ==========================================
		  	// 场景：John 学会了法语，要加进去
		  	fr := Language{Name: "French"}
		  	
		  	// 使用 Association 模式（推荐）
		  	// 这里的 "Languages" 对应结构体字段名
		  	err := db.Model(&u).Association("Languages").Append(&fr)
		  	if err == nil {
		  		fmt.Println("已添加法语关联")
		  	}
		  
		  	// ==========================================
		  	// 4. 删除关联 (Delete Association)
		  	// ==========================================
		  	// 场景：John 忘记了英语 (只删除关系，不删除英语这个数据本身)
		  	// 注意：这里需要传入具体的对象，或者对象的 ID
		  	db.Model(&u).Association("Languages").Delete(&en)
		  	fmt.Println("已移除英语关联")
		  
		  	// ==========================================
		  	// 5. 替换关联 (Replace)
		  	// ==========================================
		  	// 场景：重置，John 现在只懂日语
		  	jp := Language{Name: "Japanese"}
		  	db.Model(&u).Association("Languages").Replace(&jp)
		  	fmt.Println("关联已重置为仅日语")
		  }
		  ```
	- **CRUD 中间表：**
	  collapsed:: true
		- 专门用于管理关联。
		- ```go
		  // 1. 添加关系 (Append)
		  db.Model(&user).Association("Languages").Append(&newLang)
		  
		  // 2. 移除关系 (Delete) - 仅删中间表记录，不删 Language 表记录
		  db.Model(&user).Association("Languages").Delete(&oldLang)
		  
		  // 3. 替换关系 (Replace) - 先清空该用户的所有旧关系，再添加新的
		  db.Model(&user).Association("Languages").Replace(&lang1, &lang2)
		  
		  // 4. 清空关系 (Clear) - 删掉该用户在中间表的所有记录
		  db.Model(&user).Association("Languages").Clear()
		  
		  // 5. 计数 (Count) - 该用户会几种语言？
		  db.Model(&user).Association("Languages").Count()
		  ```
	- **自定义外键和中间表列名：**
	  collapsed:: true
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
	- **多对多的字段加在哪一边？**
	  collapsed:: true
		- **只查一边**：放在“主”的一方（通常是 User）。
		- **两边都要查**：两边都加，但中间表名字必须一致。
		- **数据库层面**：无论你加在一边还是两边，GORM 在数据库里生成的中间表都是同一张。
-