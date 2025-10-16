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
- **`Update` 方法：**
	- 如果你只需要更新某个字段的值，可以使用 `Update` 方法。
	- `Update` 方法有一个前提条件，即必须提供条件（`WHERE` 子句），否则会抛出错误 `ErrMissingWhereClause`。
	- **更新单个字段：**
		- **基本用法：**
			- ```go
			  // 更新 name 字段
			  db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
			  
			  UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;
			  ```
		- **使用 `Model` 和主键：**
			- 如果你使用了 `Model` 并且给定了一个具有主键的对象，GORM 会自动使用该主键构建更新条件。
			- ```go
			  // 如果 user.ID = 111
			  db.Model(&user).Update("name", "hello")
			  
			  UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;
			  ```
		- **使用条件和模型：**
			- 你还可以结合 `Model` 和 `Where` 条件来进行更精确的更新。
			- ```go
			  db.Model(&user).Where("active = ?", true).Update("name", "hello")
			  
			  UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
			  ```
	- **更新多个字段：**
		- **使用结构体（`struct`）更新：**
			- 当你需要更新多个字段时，可以通过传入一个结构体来实现。需要注意的是，使用结构体更新时，只有非零字段会被更新。
			- ```go
			  // 使用结构体更新多个字段（只有非零字段会更新）
			  db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
			  
			  UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;
			  ```
			- **注意**：如果结构体中的某些字段是零值（比如 `Age` 字段为 0，`Active` 为 `false`），这些字段将不会被更新。这是 GORM 的默认行为。
			- **使用 `Select` 和结构体（`struct`）更新：**
				- ```go
				  db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
				  // 生成的 SQL: UPDATE users SET name='new_name', age=0 WHERE id=111;
				  // 在这个例子中，Select("Name", "Age") 会更新 Name 和 Age 字段，即使 Age 字段的值是零。
				  
				  db.Model(&user).Select("*").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})
				  // 生成的 SQL: UPDATE users SET name='jinzhu', role='admin', age=0 WHERE id=111;
				  // 这里 Select("*") 表示选择所有字段并更新，包括零值字段。
				  
				  db.Model(&user).Select("*").Omit("Role").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})
				  // 生成的 SQL: UPDATE users SET name='jinzhu', age=0 WHERE id=111;
				  // Omit("Role") 会忽略 Role 字段的更新，只更新 Name 和 Age 字段。
				  ```
		- **使用 `map` 更新：**
			- 如果你希望更新所有字段（包括零值字段），你可以使用 `map` 来指定字段和值。通过 `map` 传入的所有字段都会被更新（无论字段的值是否为零值）。
			- ```go
			  // 使用 map 更新多个字段（包括零值字段）
			  db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
			  
			  UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
			  ```
	- **更新选定字段：**
		- 在 GORM 中，除了基本的更新操作，我们还可以选择只更新部分字段或者忽略某些字段。这可以通过 `Select` 和 `Omit` 来实现。
		- `Select`：用于指定更新哪些字段。
		- `Omit`：用于指定忽略某些字段。
		- **使用 `Select` 更新特定字段：**
			- ```go
			  // 假设用户的 ID 是 111
			  db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
			  // 生成的 SQL: UPDATE users SET name='hello' WHERE id=111;
			  ```
		- **使用 `Omit` 忽略字段：**
			- ```go
			  db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
			  // 生成的 SQL: UPDATE users SET age=18, active=false WHERE id=111;
			  ```
- **更新钩子：**
	- **BeforeUpdate**：在更新操作之前执行。
	- **AfterUpdate**：在更新操作之后执行。
- **批量更新：**
	- **使用结构体进行批量更新：**
		- ```go
		  db.Model(User{}).Where("role = ?", "admin").Updates(User{Name: "hello", Age: 18})
		  // 生成的 SQL: UPDATE users SET name='hello', age=18 WHERE role = 'admin';
		  ```
	- **使用 `map` 进行批量更新：**
		- ```go
		  db.Table("users").Where("id IN ?", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
		  // 生成的 SQL: UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);
		  ```
- **阻止全局更新：**
	- GORM 默认不允许全局更新，如果没有提供任何条件，GORM 会抛出 `ErrMissingWhereClause` 错误。
	- **启用全局更新：**
		- 如果你想允许没有条件的全局更新，可以通过设置 `AllowGlobalUpdate` 来启用：
		- ```go
		  db.Session(&gorm.Session{AllowGlobalUpdate: true}).Model(&User{}).Update("name", "jinzhu")
		  // 生成的 SQL: UPDATE users SET name='jinzhu';
		  ```
	- **直接执行原始 SQL 更新：**
		- ```go
		  db.Exec("UPDATE users SET name = ?", "jinzhu")
		  // 生成的 SQL: UPDATE users SET name = "jinzhu";
		  ```
	- **使用条件进行全局更新：**
		- 如果需要执行全局更新，必须明确指定条件。
		- ```go
		  db.Model(&User{}).Where("1 = 1").Update("name", "jinzhu")
		  // 生成的 SQL: UPDATE users SET name='jinzhu' WHERE 1=1;
		  // 这里 Where("1 = 1") 是一个始终为 true 的条件，相当于更新表中所有记录。
		  ```
-