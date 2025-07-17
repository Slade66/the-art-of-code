- **删除单条记录：**
	- 删除单条记录时，需要提供该记录的主键。没有主键的话，GORM 会执行批量删除操作。
	- ```go
	  db.Delete(&Email{ID: 10})
	  // 生成的 SQL： DELETE FROM emails WHERE id = 10;
	  
	  db.Where("name = ?", "jinzhu").Delete(&Email{})
	  // 生成的 SQL： DELETE FROM emails WHERE name = "jinzhu";
	  
	  db.Delete(&User{}, "age = ?", 30)
	  // 生成的 SQL： DELETE FROM users WHERE age = 30;
	  ```
- **批量删除：**
	- ```go
	  db.Delete(&users, []int{1, 2, 3})
	  // 生成的 SQL： DELETE FROM users WHERE id IN (1, 2, 3);
	  
	  db.Where("email LIKE ?", "%jinzhu%").Delete(&Email{})
	  // 生成的 SQL： DELETE FROM emails WHERE email LIKE "%jinzhu%";
	  
	  db.Delete(&Email{}, "email LIKE ?", "%jinzhu%")
	  // 生成的 SQL： DELETE FROM emails WHERE email LIKE "%jinzhu%";
	  ```
- **软删除：**
	- 软删除的概念是指不会从数据库中真正删除记录，而是通过标记将其标识为已删除。GORM 默认支持软删除，只需在模型中添加 `gorm.DeletedAt` 字段即可。只要在模型中包含了 `gorm.DeletedAt` 字段，GORM 会自动处理软删除的相关逻辑。
	- 如果你的模型中包含了 `gorm.DeletedAt` 字段，GORM 会自动将删除操作转换为软删除（即不会真正删除记录，而是将 `DeletedAt` 字段设置为当前时间）。
	- **查询被软删除记录：**
		- 默认情况下，软删除的记录不会被查询出来。要查询软删除的记录，可以使用 `Unscoped()`：
		- ```go
		  db.Unscoped().Where("age = 20").Find(&users)
		  // 生成的 SQL： SELECT * FROM users WHERE age = 20 AND deleted_at IS NOT NULL;
		  ```
- **永久删除：**
	- 使用 `Unscoped()` 后，`Delete` 会执行真正的删除操作，将记录从数据库中完全移除。
	- ```go
	  db.Unscoped().Delete(&Order{ID: 10})
	  // 生成的 SQL： DELETE FROM orders WHERE id = 10;
	  ```
- **返回删除的数据：**
	- 可以使用 `Returning` 子句来实现在删除记录时返回已删除的数据。
	- ```go
	  // 返回所有列的删除数据：
	  var users []User
	  DB.Clauses(clause.Returning{}).Where("role = ?", "admin").Delete(&users)
	  // 生成的 SQL： DELETE FROM users WHERE role = "admin" RETURNING *;
	  // users => [{ID: 1, Name: "jinzhu", Role: "admin"}, {ID: 2, Name: "jinzhu.2", Role: "admin"}]
	  
	  // 返回指定列的删除数据：
	  DB.Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Delete(&users)
	  // 生成的 SQL： DELETE FROM users WHERE role = "admin" RETURNING name, salary;
	  // users => [{ID: 0, Name: "jinzhu", Salary: 100}, {ID: 0, Name: "jinzhu.2", Salary: 1000}]
	  ```
-