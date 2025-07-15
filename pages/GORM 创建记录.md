- **创建单条记录：**
	- 要创建一条记录，你首先需要初始化一个包含数据的 Go struct 对象，然后将该对象的指针传递给 `db.Create()` 方法。
	- **代码示例：**
		- ```go
		  // 1. 初始化一个 User 对象
		  user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
		  
		  // 2. 将指针传递给 Create 方法
		  result := db.Create(&user)
		  
		  // 3. 获取返回结果
		  fmt.Println("新插入记录的 ID:", user.ID)        // GORM 会自动将生成的主键回填到 user 对象
		  fmt.Println("错误信息:", result.Error)          // 如果有错误，会在这里返回，没有就是 nil
		  fmt.Println("影响的行数:", result.RowsAffected) // 返回插入的记录数，这里是 1
		  ```
	- **为什么是传递指针？**
		- 因为 GORM 在创建记录成功后，需要将数据库自动生成的主键（比如 `ID`）和其它值（比如 `CreatedAt` 时间）写回到你传入的那个 struct 对象中，所以需要指针才能修改它。
- **创建多条记录：**
	- 如果你想一次性插入多条数据，只需要将一个 struct 的切片（slice）传递给 `Create` 方法即可。GORM 会智能地将它们打包成一条 SQL 语句（如果数据库支持的话），效率更高。
	- **代码示例：**
		- ```go
		  // 准备一个用户切片
		  users := []User{
		    {Name: "Jinzhu", Age: 18, Birthday: time.Now()},
		    {Name: "Jackson", Age: 19, Birthday: time.Now()},
		    {Name: "Peter", Age: 20, Birthday: time.Now()},
		  }
		  
		  // 传递切片给 Create
		  result := db.Create(&users) // 注意，这里也是传递 users 的指针，以便 GORM 回填 ID
		  
		  // 检查结果
		  for _, u := range users {
		    fmt.Println("新用户 ID:", u.ID) // 每个用户的 ID 都会被回填
		  }
		  fmt.Println("错误信息:", result.Error)
		  fmt.Println("影响的行数:", result.RowsAffected) // 返回 3
		  ```
- **指定字段创建：**
	- 有时你只想创建或更新模型中的某几个字段，而不是全部字段。
	- `Select`：白名单模式，只包含你指定的字段。
	- `Omit`：黑名单模式，排除你指定的字段。
	- **代码示例：**
		- ```go
		  user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
		  
		  // 只创建 Name 和 Age 字段，其他字段（如 Birthday）将被忽略
		  db.Select("Name", "Age").Create(&user)
		  // 生成的 SQL: INSERT INTO `users` (`name`,`age`) VALUES ("Jinzhu", 18)
		  
		  // 创建时忽略 Name 和 Age 字段，只创建其他字段
		  db.Omit("Name", "Age").Create(&user)
		  // 生成的 SQL: INSERT INTO `users` (`birthday`, ...) VALUES ("2025-07-15 ...", ...)
		  ```
- **创建钩子：**
	- 钩子（Hooks）是在进行增删改查操作时，能够自动触发执行的函数。对于创建操作，有两个常用的钩子：
		- `BeforeCreate`: 在记录插入数据库之前执行。
		- `AfterCreate`: 在记录成功插入数据库之后执行。
	- 你只需要在你的模型 struct 上实现这些方法即可。
	- **代码示例：**
		- ```go
		  import "github.com/google/uuid"
		  
		  // 在 User 模型上实现 BeforeCreate 钩子
		  func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
		    // 在创建前自动生成一个 UUID
		    // 假设 User struct 中有一个 UUID 字段
		    // u.UUID = uuid.New().String()
		  
		    fmt.Printf("准备创建用户: %s\n", u.Name)
		    // 你也可以在这里进行数据验证
		    if u.Age < 18 {
		      return errors.New("未成年人禁止注册")
		    }
		    return nil
		  }
		  
		  // ...
		  user := User{Name: "Underage", Age: 17}
		  result := db.Create(&user) // 这里会失败，因为 BeforeCreate 返回了错误
		  fmt.Println(result.Error) // 输出: 未成年人禁止注册
		  ```
	- 如果你想在某次操作中跳过钩子，可以使用 `SkipHooks`：
		- ```go
		  db.Session(&gorm.Session{SkipHooks: true}).Create(&user)
		  ```
- **从 Map 创建：**
	- 你可以直接从 `map[string]interface{}` 创建数据。
	- **代码示例：**
		- ```go
		  // 注意：使用 map 创建时需要先用 Model() 指定要操作的表
		  db.Model(&User{}).Create(map[string]interface{}{
		    "Name": "jinzhu_from_map", "Age": 25,
		  })
		  
		  // 批量插入
		  db.Model(&User{}).Create([]map[string]interface{}{
		    {"Name": "user1", "Age": 18},
		    {"Name": "user2", "Age": 20},
		  })
		  ```
	- 从 map 创建时，不会触发钩子，不会保存关联数据，也不会回填主键。
- **创建记录时同时创建关联的记录：**
	- 如果你的模型之间有关联（如 `Has One`, `Has Many`），在创建主对象时，GORM 可以自动创建或更新其关联的对象。
	- ```go
	  type CreditCard struct {
	    gorm.Model
	    Number   string
	    UserID   uint
	  }
	  
	  type User struct {
	    gorm.Model
	    Name       string
	    CreditCard CreditCard // Has One 关联
	  }
	  
	  // 创建 User 的同时，也为他创建一张 CreditCard
	  user := User{
	    Name: "jinzhu",
	    CreditCard: CreditCard{Number: "4111-1111-1111-1111"},
	  }
	  db.Create(&user)
	  // GORM 会执行两条 INSERT 语句，一条给 users 表，一条给 credit_cards 表
	  ```
	- 如果你想跳过关联，可以使用 `Omit`：
		- ```go
		  // Omit 单个关联
		  db.Omit("CreditCard").Create(&user)
		  
		  // Omit 所有关联
		  db.Omit(clause.Associations).Create(&user)
		  ```
- **插入或更新：**
	- 如果记录不存在，则插入；如果记录已存在（通常根据主键或唯一索引判断），则更新它。
	- ```go
	  import "gorm.io/gorm/clause"
	  
	  // 准备数据，假设 id=1 的用户已存在
	  users := []User{{ID: 1, Name: "old_name"}, {ID: 2, Name: "new_user"}}
	  
	  // 1. 冲突时什么都不做 (Do Nothing)
	  db.Clauses(clause.OnConflict{DoNothing: true}).Create(&users)
	  // 结果: id=1 的记录不变，id=2 的记录被插入
	  
	  // 2. 冲突时更新指定字段为新值
	  db.Clauses(clause.OnConflict{
	    Columns:   []clause.Column{{Name: "id"}}, // 指定冲突键
	    DoUpdates: clause.AssignmentColumns([]string{"name", "age"}), // 更新 name 和 age 字段
	  }).Create(&users)
	  // 结果: id=1 的记录 name 被更新，id=2 的记录被插入
	  
	  // 3. 冲突时更新所有字段为新值
	  db.Clauses(clause.OnConflict{
	    UpdateAll: true,
	  }).Create(&users)
	  // 结果: id=1 的记录所有字段都被更新，id=2 的记录被插入
	  ```
- **分批插入：**
	- `CreateInBatches` 用于一次性将多个记录插入到数据库中，减少数据库交互次数，避免逐条插入带来的性能瓶颈。
	- `CreateInBatches` 的底层原理：
		- 启动事务
		  logseq.order-list-type:: number
			- GORM 首先会开启一个数据库事务。这是至关重要的一步，它保证了整个批量插入操作的原子性。也就是说，要么所有批次的数据都成功插入，要么在任何一个批次失败时，所有已经插入的数据都会被回滚，数据库将恢复到操作之前的状态，避免了只插入部分数据的“脏”状态。
		- 数据分片
		  logseq.order-list-type:: number
			- GORM 会像切蛋糕一样，将你传入的巨大切片（比如 10,000 条 `users`）按照你指定的批次大小（`100`）进行切割。
		- 循环执行插入
		  logseq.order-list-type:: number
			- GORM 会遍历这些“小批次”的数据。在每次循环中，它会为当前批次（例如 100 条记录）生成一条复合 `INSERT` 语句（`VALUES`）。然后在同一个事务中，执行这条 SQL 语句。
		- 提交事务
		  logseq.order-list-type:: number
			- 如果所有批次的 `INSERT` 操作都成功执行，没有任何错误，GORM 最终会提交（Commit）这个事务，将所有数据永久性地保存在数据库中。
-