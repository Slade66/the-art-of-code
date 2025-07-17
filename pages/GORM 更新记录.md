- **`Save` 方法：**
	- 用于保存记录到数据库。它在执行数据库操作时，既可以进行更新（`Update`），也可以进行插入（`Insert`）。这个行为被称为 upsert（即：更新或插入）。
	- **什么是 Upsert？**
		- Upsert 是 "Update" 和 "Insert" 的合成词，意味着：
			- 如果记录已经存在（通过主键确定），则执行更新（`Update`）操作。
			- 如果记录不存在，则执行插入（`Insert`）操作。
	- **`Save` 方法的行为：**
		- **如果没有主键**：`Save` 会执行插入操作。
		- **如果有主键**：`Save` 会先执行更新（`Update`）操作。更新时，所有字段都会被更新。如果更新操作没有影响到任何行（例如没有找到对应的记录），GORM 会自动回退到插入操作。
		- **更新所有字段：**当使用 `Save` 方法进行更新时，它会将记录中的所有字段都更新到数据库中。即使某些字段没有变化，`Save` 也会将它们一起更新。
	- **代码示例：**
		- ```go
		  // 定义用户模型
		  type User struct {
		  	ID        uint   `gorm:"primary_key"`
		  	Name      string
		  	Age       int
		  	Birthday  string
		  	UpdatedAt string
		  }
		  
		  // 插入新用户
		  newUser := User{Name: "Alice", Age: 25}
		  db.Save(&newUser)  // 执行插入操作
		  
		  // 更新现有用户
		  existingUser := User{ID: 1, Name: "Bob", Age: 30}
		  db.Save(&existingUser)  // 执行更新操作
		  
		  // Upsert 操作（根据 ID 更新或插入）
		  upsertUser := User{ID: 2, Name: "Charlie", Age: 35}
		  db.Save(&upsertUser)  // 如果 ID=2 的记录存在，执行更新，否则执行插入
		  
		  // 插入没有 ID 的新用户
		  noIDUser := User{Name: "David", Age: 40}
		  db.Save(&noIDUser)  // 执行插入操作
		  ```
-