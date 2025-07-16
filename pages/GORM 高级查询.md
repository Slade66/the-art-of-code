- **Group 条件：**
	- GORM 通过代码中的嵌套结构（`Where` 套着 `Where/Or`），构建带有复杂括号 `()` 的 `WHERE` 查询，确保 `AND` 和 `OR` 操作的执行顺序正确，帮助理清复杂的“和”与“或”逻辑，避免电脑误解你的意图。
	- **比喻：**就像你在网上点外卖，你的要求是：“主食是汉堡，并且饮料是（可乐 或 雪碧）”。这个括号很重要，如果没括号，意思就可能变成“(主食是汉堡并且饮料是可乐) 或 雪碧”，那就全乱了。
	- ```go
	  condition1 := db.Where("product_name = ?", "高端游戏本").
	      Where(db.Where("status = ?", "pending").Or("status = ?", "processing"))
	  
	  condition2 := db.Where("product_name = ?", "机械键盘").
	      Where("amount > ?", 200)
	  
	  var orders []Order
	  db.Where(condition1).Or(condition2).Find(&orders)
	  
	  // SELECT * FROM `orders` WHERE
	  // (product_name = '高端游戏本' AND (status = 'pending' OR status = 'processing'))
	  // OR (product_name = '机械键盘' AND amount > 200)
	  ```
- **带多个列的 In：**
	- **作用：** 直接将你需要的“组合”条件列表交给 GORM，它会一次性匹配多个“CP组合”的数据。
	- **比喻：**你想找两对情侣，一对是“罗密欧和朱丽叶”，另一对是“梁山伯和祝英台”。你不想分开找两次，想一次性告诉系统：“把这两对都给我找出来”。
	- ```go
	  // 找出 (用户A买的商品X) 和 (用户B买的商品Y) 这两笔订单
	  pairs := [][]interface{}{ {"用户A", "商品X"}, {"用户B", "商品Y"} }
	  db.Where("(user_name, product_name) IN ?", pairs).Find(&orders)
	  
	  // SELECT * FROM `orders` 
	  // WHERE (user_name, product_name) IN (('用户A','商品X'),('用户B','商品Y'))
	  ```
- **子查询：**
	- **用在 `WHERE` 条件里的子查询：**
		- **用途：**用一个查询的结果，来作为另一个查询的筛选条件。
		- **比喻：**
			- 老师想表扬班里所有“成绩高于平均分”的同学。
			- 他不能直接问“谁的成绩高于平均分？”，因为他自己也不知道平均分是多少。
			- 他需要先做一个“内部计算”（子查询）：“咱们班的平均分是多少？” 得到结果，比如 `85`分。
			- 然后再进行主查询：“谁的成绩高于 `85` 分？”
		- **示例：**
			- ```go
			  var vipOrders []Order
			  // 1. 先定义那个“内部计算”的查询：计算所有订单的平均金额
			  avgAmountQuery := db.Model(&Order{}).Select("AVG(amount)")
			  
			  // 2. 在主查询的 Where 条件里，用 (?) 占位符来使用它
			  db.Where("amount > (?)", avgAmountQuery).Find(&vipOrders)
			  
			  // SELECT * FROM `orders` WHERE amount > (SELECT AVG(amount) FROM `orders`)
			  ```
			- GORM 非常聪明。当你把 `avgAmountQuery` 这个 GORM 查询对象传给 `Where` 时，它不会真的去执行它，而是把它翻译成 SQL `(SELECT AVG(amount) FROM orders)`，然后塞进 `(?)` 的位置。这就完美地实现了我们的需求。
	- **用在 `FROM` 里的子查询：**
		- `FROM` 子查询允许你将一个查询的结果作为中间表来使用，并在外层查询中引用它。
		- ```go
		  var youngActiveUsers []User
		  // 1. 先定义那张“中间表”长什么样：只包含活跃用户的名字和年龄
		  activeUsersQuery := db.Model(&User{}).Select("name", "age").Where("status = ?", "active")
		  
		  // 2. 用 Table("(?) AS ...") 来告诉 GORM，我要查询的是这张“中间表”
		  db.Table("(?) AS active_users", activeUsersQuery).Where("age < ?", 20).Find(&youngActiveUsers)
		  
		  // SELECT * FROM (SELECT `name`,`age` FROM `users` WHERE `status` = 'active')
		  // AS active_users WHERE `age` < 20
		  ```
- **智能选择字段：**
	- **作用：**按你指定的“模板”来挑数据，自动忽略不需要的字段。
	- **举例：**你有一张非常详细的个人信息表，但只想打印一张只包含“姓名”和“电话”的名片。你只需要提供一个名片的空模板，GORM 就会自动帮你从信息表里挑出这两项来填充。
	- `Find()` 的目标 `struct` 长什么样，GORM 就帮你 `SELECT` 什么，非常智能。
	- ```go
	  type UserCard struct { Name, Phone string } // 这就是名片模板
	  var cards []UserCard
	  db.Model(&User{}).Find(&cards) // GORM一看你要的是名片，就只去查 Name 和 Phone
	  
	  // SELECT `name`, `phone` FROM `users`
	  ```
- **Find 至 map：**
	- 如果你只是临时查看某个商品的信息，又不想为结果集专门创建一个 `struct`，那么随手拿个塑料袋（`map`），把所有信息一股脑装进去。
	- `map` 就是万能容器，什么都能装，最适合处理动态或临时的查询。
	- ```go
	  var productBag map[string]interface{}
	  db.Model(&Product{}).First(&productBag, 101)
	  
	  // SELECT * FROM `products` WHERE `id` = 101 LIMIT 1
	  ```
- **Pluck：**
	- **作用：**如果你只需要一整列数据，使用 `Pluck` 可以轻松地将单列数据提取出来。
	- **举例：**全班同学都填了身高，你现在只想要一个“身高列表”，对其他信息（姓名、年龄）都不感兴趣。`Pluck` 就像一只手，直接把“身高”那一列数据抽出来。
	- ```go
	  var names []string
	  db.Model(&User{}).Pluck("name", &names)
	  
	  // SELECT `name` FROM `users`
	  ```
- **FirstOrInit：**
	- **作用：**先去数据库里找，找不到就用预先准备的默认数据。只在程序里用，不存数据库。
	- ```go
	  // 找叫 "jinzhu" 的用户，找不到就初始化一个叫 "jinzhu" 的空用户
	  db.FirstOrInit(&user, User{Name: "jinzhu"})
	  
	  // SELECT * FROM `users` WHERE `name` = 'jinzhu' LIMIT 1
	  ```
- **FirstOrCreate：**
	- **作用：**先去数据库里找，找不到就新建一个。真的会存到数据库里。
	- ```go
	  // 找叫 "jinzhu" 的用户，找不到就新建一个
	  db.FirstOrCreate(&user, User{Name: "jinzhu"})
	  
	  -- 第一步: 查找
	  // SELECT * FROM `users` WHERE `name` = 'jinzhu' LIMIT 1;
	  -- 如果上面没找到，会执行第二步:
	  // INSERT INTO `users` (`name`) VALUES ('jinzhu');
	  ```
- **锁：**
	- **用途：**在很多人同时抢一件东西时，给这件东西“上个锁”，保证一次只有一个人能操作它，防止数据错乱。
	- **排他锁（`FOR UPDATE`）：**
		- `FOR UPDATE` 就是那个“谁都别碰”的信号。一个事务只要对某行数据用了它，其他任何想修改或读取这行数据的事务都必须排队，直到当前事务结束。
		- ```go
		  db.Transaction(func(tx *gorm.DB) error {
		      var ticket Ticket
		      // 1. 查找门票ID为1的记录，并立即上锁
		      if err := tx.Clauses(clause.Locking{Strength: "UPDATE"}).First(&ticket, 1).Error; err != nil {
		          return err
		      }
		  
		      // 2. 检查库存
		      if ticket.Stock < 1 {
		          return errors.New("没票了！")
		      }
		  
		      // 3. 减库存并保存
		      ticket.Stock -= 1
		      return tx.Save(&ticket).Error
		      // 4. 事务到这里结束，锁自动解开
		  })
		  
		  // SELECT * FROM `tickets` WHERE `id` = 1 FOR UPDATE
		  ```
	- **共享锁（`FOR SHARE`）：**
		- `FOR SHARE` 允许多个事务同时读取同一份数据（大家一起看），但任何想修改（`FOR UPDATE`）这份数据的事务都必须等所有读取的事务结束。它保证了你在读取期间，数据不会被篡改。
		- ```go
		  var stock int
		  db.Model(&Ticket{}).Clauses(clause.Locking{Strength: "SHARE"}).Pluck("stock", &stock)
		  
		  // SELECT `stock` FROM `tickets` FOR SHARE
		  ```
	- **`NOWAIT` 选项：**
		- `NOWAIT` 能防止你的程序因为等待锁而被长时间卡住（阻塞）。如果获取不到锁，它不会等待，而是立刻返回一个错误，让你的程序可以去做别的事情。
		- ```go
		  err := db.Clauses(clause.Locking{Strength: "UPDATE", Options: "NOWAIT"}).First(&ticket, 1).Error
		  
		  // SELECT * FROM `tickets` WHERE `id` = 1 FOR UPDATE NOWAIT
		  ```
	- **`SKIP LOCKED` 选项：**
		- `SKIP LOCKED` 在处理任务队列时是神器。比如有10个后台程序同时去处理1000个待办任务，每个程序都用 `SKIP LOCKED` 去取任务，它们就能自动领走不同的任务而不会互相等待，大大提高了并行处理效率。
		- **比喻：**你是一个黄牛机器人，任务是抢购所有尚未被预订的票。你对系统说：“请列出所有可购买的票，如果某张票已经被加到购物车（已锁定），就跳过它，直接给我下一张可购买的票。”
		- ```go
		  // 这个场景通常用在后台任务处理中
		  var availableTickets []Ticket
		  db.Clauses(clause.Locking{Strength: "UPDATE", Options: "SKIP LOCKED"}).Limit(10).Find(&availableTickets)
		  
		  // SELECT * FROM `tickets` FOR UPDATE SKIP LOCKED LIMIT 10
		  ```
- **命名参数：**
	- **作用：**给查询里的 `?` 占位符起个名字，让代码更像人话。
	- **比喻：**
		- 想象一下，你正在填写一份很长的表格，里面有很多空白格需要你填。
		- **传统方式（`?`占位符）：**说明书上写着：“请在第1空、第5空、第8空都填上你的‘姓名’”。你得一边数着空位，一边填，生怕数错了。如果你要填的值很多，比如姓名、地址、电话，你得非常小心地按顺序 `("张三", "某某路", "138...", "张三")` 这样提供，顺序一乱就全错了。
		- **命名参数（`@name`）：**说明书换了一种更友好的方式：“请在所有标有`【姓名】`的地方，都填上你的姓名”。这就简单多了！你只需要知道 `【姓名】 = "张三"`，然后系统会自动帮你把所有标有`【姓名】`的空都填好。
		- 命名参数就是 GORM 里的第二种方式，它用 `@` 符号来创建带名字的“空”。
	- **示例：**
		- 我们要做一个全局搜索，当用户输入一个关键词后，我们需要在 `products` 表的 `name` (商品名) 和 `description` (商品描述) 两个字段里都查找这个关键词。
		- ```go
		  var products []Product
		  keyword := "无线"
		  // @keyword 就是我们起的名字
		  db.Where("name LIKE @keyword OR description LIKE @keyword",
		      map[string]interface{}{"keyword": "%" + keyword + "%"},
		  ).Find(&products)
		  
		  -- GORM/数据库驱动在后台会将命名参数转换为数据库能识别的格式
		  SELECT * FROM `products` WHERE name LIKE '%无线%' OR description LIKE '%无线%'
		  ```
-