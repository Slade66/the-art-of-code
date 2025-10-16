- **GORM 的默认行为：**
	- 为了保证数据安全，GORM 默认会为每一次单独的写操作（`Create`, `Update`, `Delete`）自动包裹一个事务。例如，当你执行 `db.Create(&user)` 时，GORM 实际执行的是 `BEGIN; CREATE ...; COMMIT;`。
- **`db.Transaction`（自动事务）：**
	- 你只需要提供一个闭包函数，GORM 会自动处理 `Begin`, `Commit`, `Rollback` 的所有逻辑。
	- **规则非常简单：**
		- 如果你的函数返回 `nil` (没有错误)，GORM 会自动 `Commit` 事务。
		- 如果你的函数返回任何 `error`，GORM 会自动 `Rollback` 事务。
	- ```go
	  db.Transaction(func(tx *gorm.DB) error {
	    // 重点：从这里开始，所有数据库操作都必须使用 tx 这个句柄，而不是外面的 db
	    
	    // 尝试创建一个 "Giraffe"
	    if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
	      // 返回错误，整个事务将回滚
	      return err
	    }
	  
	    // 尝试创建一个 "Lion"
	    if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
	      // 返回错误，事务同样会回滚。"Giraffe" 也会被一并回滚掉。
	      return err
	    }
	  
	    // 函数成功结束，返回 nil，GORM 将提交整个事务
	    return nil
	  })
	  ```
	- 推荐用于绝大多数业务场景，代码简洁，不易出错，但灵活性较低。
- **手动控制事务：**
	- 如果你需要更灵活地控制事务流程，GORM 也允许你手动开启、提交和回滚事务。
	- **流程是：**`Begin` -> 执行操作 -> `Commit` 或 `Rollback`。
	- ```go
	  func CreateAnimals(db *gorm.DB) error {
	    // 开启事务
	    tx := db.Begin()
	    
	    // 使用 defer 和 recover 来捕获未预料的 panic，确保即使程序崩溃也能回滚事务
	    defer func() {
	      if r := recover(); r != nil {
	        tx.Rollback()
	      }
	    }()
	  
	    // 检查开启事务本身是否出错
	    if err := tx.Error; err != nil {
	      return err
	    }
	  
	    // 执行具体操作，每次操作后都检查错误
	    if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
	       tx.Rollback() // 出错了，回滚并返回错误
	       return err
	    }
	  
	    if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
	       tx.Rollback() // 出错了，回滚并返回错误
	       return err
	    }
	  
	    // 所有操作都成功，提交事务，并返回提交操作本身可能产生的错误
	    return tx.Commit().Error
	  }
	  ```
- **嵌套事务：**
	- 你可以在一个大事务中开启一个“子事务”。子事务的回滚不会影响到父事务。
	- ```go
	  db.Transaction(func(tx *gorm.DB) error {
	    // 父事务中创建 user1
	    tx.Create(&user1)
	  
	    // 开启一个子事务
	    tx.Transaction(func(tx2 *gorm.DB) error {
	      tx2.Create(&user2)
	      return errors.New("rollback user2") // 返回错误，这个子事务回滚，user2 不会被创建
	    }) // 子事务结束
	  
	    // 开启另一个子事务
	    tx.Transaction(func(tx3 *gorm.DB) error {
	      tx3.Create(&user3)
	      return nil // 返回 nil，这个子事务提交，user3 会被创建
	    }) // 子事务结束
	  
	    return nil // 父事务返回 nil
	  })
	  // 最终结果：user1 和 user3 被成功创建并提交，user2 因为子事务回滚而未被创建。
	  ```
- **保存点：**
	- `SavePoint` 就像在游戏中设置一个“存档点”。你可以在事务的任何地方创建一个命名保存点，之后如果出现问题，你可以选择回滚到这个点，而不是回滚整个事务。
	- ```go
	  tx := db.Begin()
	  tx.Create(&user1) // user1 已进入事务
	  
	  tx.SavePoint("sp1") // 创建一个名为 "sp1" 的存档点
	  
	  tx.Create(&user2) // user2 已进入事务
	  
	  tx.RollbackTo("sp1") // 出现问题，回滚到 "sp1" 点。这会撤销 user2 的创建，但 user1 还在。
	  
	  tx.Commit() // 提交事务
	  // 最终结果：只有 user1 被成功创建。
	  ```
-