- `HAS ONE` 关系表示一个模型实体拥有另一个模型实体的单个实例。
- **比喻：**
	- 一个经典的例子是：一个用户 (`User`) 拥有一张信用卡 (`CreditCard`)。
	- `User` 是所有者，`CreditCard` 是被拥有的对象。
	- 它们之间的关联通过外键 (Foreign Key) 实现。外键字段通常存放在被拥有对象所在的数据库表中。在这个例子里，`credit_cards` 表会有一个 `user_id` 字段，用来标识信用卡属于哪个用户。
	- 为什么外键放在被拥有对象上呢？就像银行卡上会写你的名字，但你身上不会写银行卡的卡号。
- **定义 HAS ONE 模型：**
	- 在所有者 struct 中嵌入被拥有者的 struct，在被拥有者 struct 中定义外键字段（如 `UserID`）。
	- ```go
	  // User 模型
	  type User struct {
	      gorm.Model // 内嵌 gorm.Model，包含 ID, CreatedAt, UpdatedAt, DeletedAt
	      Name       string
	      // User 有一张 CreditCard，这是 HAS ONE 关系
	      // GORM 会根据 `CreditCard` 结构体中的 `UserID` 字段来识别外键
	      CreditCard CreditCard 
	  }
	  
	  // CreditCard 模型
	  type CreditCard struct {
	      gorm.Model
	      Number string
	      // 这是外键，GORM 默认会使用所有者模型的类型名 + "ID" 作为外键名
	      // 即 User + ID -> UserID
	      UserID uint 
	  }
	  ```
- **基本操作：**
	- **创建：**
		- ```go
		  // 创建一个用户，并同时为他创建一张信用卡
		  user := User{
		      Name: "Alice",
		      CreditCard: CreditCard{ // 嵌套初始化
		          Number: "1234-5678-9012-3456",
		      },
		  }
		  
		  // GORM 会开启一个事务，先创建 User，然后获取 User 的 ID，
		  // 再将这个 ID 赋值给 CreditCard 的 UserID 字段，最后创建 CreditCard。
		  db.Create(&user)
		  
		  // 执行的 SQL (伪代码):
		  // BEGIN;
		  // INSERT INTO `users` (`name`, ...) VALUES ("Alice", ...);
		  // INSERT INTO `credit_cards` (`number`, `user_id`, ...) VALUES ("1234-...", LAST_INSERT_ID(), ...);
		  // COMMIT;
		  ```
	- **查询：**
		- 默认情况下，当你查询一个 `User` 时，GORM 不会自动加载其关联的 `CreditCard`。这是为了性能考虑，避免不必要的数据库查询。你需要使用 `Preload` 来显式地加载关联数据。
		- ```go
		  var retrievedUser User
		  // 1. 查找 ID 为 1 的用户，但不加载信用卡信息
		  db.First(&retrievedUser, 1)
		  // retrievedUser.CreditCard.ID 会是 0 (空)
		  
		  // 2. 使用 Preload 预加载 CreditCard 信息
		  db.Preload("CreditCard").First(&retrievedUser, 1)
		  // 现在 retrievedUser.CreditCard 就包含了对应的信用卡信息
		  
		  // 也可以 Preload 所有查询结果
		  var users []User
		  db.Preload("CreditCard").Find(&users)
		  // users 列表中的每个 user 都会加载其对应的 CreditCard
		  ```
-