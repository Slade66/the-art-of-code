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
	- **作用：**先去数据库找，找到了就拿来用；如果找不到，就在程序里给你准备一个新的，但注意，这个新的不会存进数据库。
	- ```go
	  // 找叫 "jinzhu" 的用户，找不到就初始化一个叫 "jinzhu" 的空用户
	  db.FirstOrInit(&user, User{Name: "jinzhu"})
	  
	  // SELECT * FROM `users` WHERE `name` = 'jinzhu' LIMIT 1
	  ```
	- `Attrs`：
		- **作用：**只在数据库里找不到记录，需要初始化一个“新”对象时，才使用这些默认值。
		- **举例：**
			- 查找一个用户，如果他不存在，就准备一个带有默认年龄的新用户对象。
			- ```go
			  var user User
			  // 查找 "jinzhu"，他已经存在了
			  db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 99}).FirstOrInit(&user)
			  // 结果: user 是数据库里那个18岁的 jinzhu，Attrs(Age: 99) 被完全忽略了。
			  
			  // 查找 "non_existing"，他不存在
			  db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrInit(&user)
			  // 结果: 找不到，所以 GORM 初始化了一个新对象，
			  // name 来自 Where("non_existing")，age 来自 Attrs(20)。
			  // 最终 user 变量是 {Name: "non_existing", Age: 20}。
			  ```
	- `Assign`：
		- **作用：**不管找没找到，都把这些值强行设置到最终查出来的 Go 变量上（但同样不存库）。
		- **举例：**
			- 查找一个用户，不管他存不存在，都在内存中把他的年龄设置为 20。
			- ```go
			  var user User
			  // 查找 "jinzhu"，他已经存在，是18岁
			  db.Where(User{Name: "jinzhu"}).Assign(User{Age: 20}).FirstOrInit(&user)
			  // 结果: 先从数据库查出18岁的 jinzhu，然后 Assign 生效，
			  // 强行将 user 变量里的 Age 改成了 20。
			  // 最终 user 变量是 {ID: 111, Name: "jinzhu", Age: 20}。
			  
			  // 查找 "non_existing"，他不存在
			  db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrInit(&user)
			  // 结果: 找不到，初始化一个新对象，
			  // name 来自 Where("non_existing")，age 来自 Assign(20)。
			  // 最终 user 变量是 {Name: "non_existing", Age: 20}。
			  ```
- **FirstOrCreate：**
	- **作用：**先去数据库找，找到了就拿来用；如果找不到，我就帮你新建一个，并且这个新的会立刻存进数据库。
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
- **优化器提示：**
	- **示例：**
		- 一个后台管理员要跑一个非常复杂的数据报表，这个查询可能会消耗大量资源，甚至拖垮整个数据库。为了安全起见，我们想限制这个查询的最长执行时间，比如30秒，超时就自动失败。
		- ```go
		  import "gorm.io/hints"
		  
		  // 使用优化器提示，来设置MySQL的最大执行时间为30000毫秒
		  db.Clauses(hints.New("MAX_EXECUTION_TIME(30000)")).Find(&ReportData{})
		  
		  SELECT * /*+ MAX_EXECUTION_TIME(30000) */ FROM `report_data`
		  ```
		- GORM 通过 `Clauses` 方法，在 SQL 语句里加上了 `/*+ ... */` 这样一段特殊的注释。这段注释虽然是注释的样子，但数据库（比如MySQL）会把它当作一个命令来解读，从而改变自己的行为。不同的数据库支持的优化器提示也不同。
- **索引提示：**
	- **作用：**直接告诉数据库该走哪条“近路”（索引）。
	- **比喻：**
		- GPS 规划了一条路线，要带你穿过市中心无数个红绿灯的小路，因为它计算出这样能快2分钟。但作为“老司机”的你知道，这条路虽然短，但极其难走。你宁愿走稍微远一点但一路通畅的高速公路。于是你强制 GPS：“别废话，就按我说的，走高速（使用 `idx_freeway` 这个索引）！”
		- ```go
		  import "gorm.io/hints"
		  
		  // 建议（USE）数据库使用 idx_user_city 这个索引
		  db.Clauses(hints.UseIndex("idx_user_city")).
		     Where("city = ? AND created_at > ?", "Los Angeles", lastMonth).
		     Find(&users)
		  
		  SELECT * FROM `users` USE INDEX (`idx_user_city`) WHERE `city` = 'Los Angeles' AND `created_at` > ...
		  
		  // 强制（FORCE）数据库在 JOIN 操作时使用某个索引
		  db.Clauses(hints.ForceIndex("idx_user_name").ForJoin()).
		     Joins("...").
		     Find(&users)
		  
		  SELECT * FROM `users` FORCE INDEX FOR JOIN (`idx_user_name`) JOIN ...
		  ```
		- `UseIndex` 像是在说：“我建议你走这条路”。
		- `ForceIndex` 更霸道，像是在说：“你必须走这条路！”
		- `ForJoin()` 是说这个索引提示只在 `JOIN` 操作时生效。
- **迭代 `Rows()`：**
	- **作用：**处理海量数据时，一行一行地从数据库吸取数据，几乎不占内存。
	- **比喻：**
		- 想象你要把一个巨大的游泳池里的水（海量数据）给抽干。
		- **普通 `Find()` 的做法**：你试图找一个和游泳池一样大的桶，想一次性把所有水都装进去。结果就是你的小卡车（内存）根本装不下，直接爆胎（内存溢出）。
		- **`Rows()` 的做法**：你只拿一根吸管。你把吸管一头放进泳池，另一头放进一个小杯子里。你一口一口地吸，吸满一小杯，处理掉，再吸下一小杯。自始至终，你手里的水（内存里的数据）都只有一小杯，所以毫不费力。`db.Rows()` 返回的 `rows` 对象，就是那根神奇的吸管。
	- **示例：**
		- 我们需要导出一个包含一百万用户信息的 CSV 文件。我们不可能把一百万个用户对象一次性加载到内存里。
		- ```go
		  // 假设 file 是一个已经打开的 csv 文件
		  csvWriter := csv.NewWriter(file)
		  csvWriter.Write([]string{"ID", "Name", "Email"}) // 写入表头
		  
		  // 1. 获取“吸管”（rows 对象），这步只发送了查询指令，并未加载数据
		  rows, err := db.Model(&User{}).Rows()
		  if err != nil { panic(err) }
		  defer rows.Close() // 非常重要：用完吸管一定要放好！
		  
		  // 2. 循环“吸水”
		  for rows.Next() {
		      var user User
		      // 3. 将当前吸到的一行数据，装进 user 这个“小杯子”
		      db.ScanRows(rows, &user)
		  
		      // 4. 处理这杯水：写入CSV文件
		      csvWriter.Write([]string{fmt.Sprint(user.ID), user.Name, user.Email})
		  }
		  csvWriter.Flush()
		  ```
		- `db.Rows()` 只会向数据库发送一次查询请求。`rows` 对象本身不存数据，它只是一个指向数据库结果集的“游标”或“指针”。`for rows.Next()` 每循环一次，这个游标就向下移动一行，然后 `db.ScanRows` 就把这一行的数据读进你的 `user` 变量里。这样，你的内存占用永远只是一个 `user` 对象的大小，和数据总量无关。
- **FindInBatches（分批处理）：**
	- **作用：**处理海量数据时，像用小推车一样，一批一批地（例如一次100条）捞数据和处理，尤其适合批量更新。
	- **比喻：**
		- 还是那个抽干游泳池的比喻。
		- 用 `Rows()` 的吸管方式，虽然稳，但如果每处理一杯水都要做点别的复杂事情，来回跑可能效率不高。
		- `FindInBatches` 就像给了你一个小推车和一个桶。你一次从泳池里装100桶水到小推车上，把这一车水推到处理区，对这100桶水进行统一加工（比如加点消毒液），然后再回去推下一车。
	- **示例：**
		- 我们需要对所有“待发货”的订单进行打包处理，处理完后需要更新它们的状态为“已打包”。我们决定一次处理100个订单。
		- ```go
		  var pendingOrders []Order
		  // 1. 每次只捞100个待发货订单
		  result := db.Where("status = ?", "pending").FindInBatches(&pendingOrders, 100, func(tx *gorm.DB, batch int) error {
		      // 2. pendingOrders 在这里就是当前这一批的100个订单
		      fmt.Printf("正在处理第 %d 批，共 %d 个订单\n", batch, len(pendingOrders))
		  
		      for i := range pendingOrders {
		          // ...执行打包、打印快递单等复杂逻辑...
		          pendingOrders[i].Status = "packed"
		      }
		  
		      // 3. 对这一整批订单，进行一次性的保存
		      return tx.Save(&pendingOrders).Error
		  })
		  
		  -- (GORM 会自动循环执行，直到捞不到数据为止)
		  -- 第1批
		  SELECT * FROM `orders` WHERE `status` = 'pending' LIMIT 100 OFFSET 0;
		  UPDATE `orders` SET `status`='packed' WHERE `id` IN (id1, id2, ...);
		  
		  -- 第2批
		  SELECT * FROM `orders` WHERE `status` = 'pending' LIMIT 100 OFFSET 100;
		  UPDATE `orders` SET `status`='packed' WHERE `id` IN (id101, id102, ...);
		  -- ...
		  ```
		- GORM 自动帮你处理了 `LIMIT` 和 `OFFSET` 的分页逻辑。你只需要在那个回调函数 `func(...)` 里，专注于处理每一批的数据就行。这种“读取-处理-写入”的循环模式，对于批量数据更新任务来说是最佳选择。
- **查询钩子：**
	- **作用：**在从数据库查出数据后，自动执行一段你预设好的“数据加工”程序。
	- **示例：**
		- 我们的 `User` 模型在数据库里存的是 `FirstName` 和 `LastName`，但程序里经常需要用到完整的 `FullName`。我们不希望在数据库里重复存 `FullName`，而是希望每次查出 `User` 后，能自动把它拼接好。
		- ```go
		  type User struct {
		      gorm.Model
		      FirstName string
		      LastName  string
		      FullName  string `gorm:"-"` // gorm:"-" 表示这个字段不需要存到数据库
		  }
		  
		  // 在 User struct 上定义这个钩子方法
		  func (u *User) AfterFind(tx *gorm.DB) (err error) {
		      // 这就是那个“自动裱花机”
		      if u.FirstName != "" {
		          u.FullName = u.FirstName + " " + u.LastName
		      }
		      return
		  }
		  
		  // 当我们查询用户时...
		  var user User
		  db.First(&user, 1)
		  // 执行完上面这行后，user.FullName 字段就已经被自动赋值了！
		  fmt.Println(user.FullName) // 输出: "Jinzhu San" (假设数据库里是 Jinzhu, San)
		  ```
		- GORM 在 `Find`、`First` 等查询操作成功后，会立刻“反思”一下：“我刚刚填充的这个 `struct`（比如 `User`），它自己有没有定义一个叫 `AfterFind` 的方法？”如果定义了，GORM 就会自动调用它。这让我们可以把一些数据后处理的逻辑，直接写在模型自己身上，非常优雅。
- **`Scope`：**
	- **作用：**把你经常用到的查询条件打包成一个“快捷方式”，方便随时调用，让代码不再重复。
	- **比喻：**
		- 想象你每天都去同一家星巴克，你每次的点单都一模一样：“大杯，冰美式，加一个shot，不要糖”。
		- **没有 Scope 的做法**：你每天都要对着店员重复念一遍这句长长的话：“你好，我想要一杯大杯的冰美式，然后麻烦加一个shot，哦对了，不要糖，谢谢。”
		- **使用 Scope 的做法**：你和熟悉的店员约定好了一个“暗号”。你对他说：“以后我只要说‘老样子’，就是指前面那句长长的话。”
	- **没有 Scope 的世界：**
		- ```go
		  // 功能点 A: 查找所有活跃用户
		  db.Where("status = ?", "active").Find(&activeUsers)
		  
		  // 功能点 B: 查找所有已上架商品
		  db.Where("is_listed = ?", true).Find(&listedProducts)
		  
		  // 功能点 C: 查找特定价格区间的已上架商品
		  db.Where("is_listed = ?", true).Where("price BETWEEN ? AND ?", 100, 500).Find(&promoProducts)
		  ```
		- **代码重复**：`Where("is_listed = ?", true)` 这个条件在 B 和 C 中都出现了一遍。如果以后“已上架”的逻辑变了（比如要加一个 `stock > 0` 的条件），你就得去改两个地方，非常容易遗漏。
	- **使用 `Scope`：**
		- `Scope` 本质上就是一个函数，它接收一个 `db` 查询对象，给它增加一些条件后，再把它返回。
		- ```go
		  // 定义一个“已上架商品”的滤镜
		  func Listed(db *gorm.DB) *gorm.DB {
		    return db.Where("is_listed = ?", true)
		  }
		  
		  // 定义一个“活跃用户”的滤镜
		  func Active(db *gorm.DB) *gorm.DB {
		    return db.Where("status = ?", "active")
		  }
		  
		  // 定义一个更灵活的、带参数的“价格区间”滤镜
		  func PriceBetween(min float64, max float64) func(db *gorm.DB) *gorm.DB {
		    return func(db *gorm.DB) *gorm.DB {
		      return db.Where("price BETWEEN ? AND ?", min, max)
		    }
		  }
		  
		  // 功能点 A: 查找所有活跃用户
		  var activeUsers []User
		  db.Model(&User{}).Scopes(Active).Find(&activeUsers)
		  
		  // 功能点 B: 查找所有已上架商品
		  var listedProducts []Product
		  db.Model(&Product{}).Scopes(Listed).Find(&listedProducts)
		  
		  // 功能点 C: 查找特定价格区间的已上架商品 (叠加使用两个滤镜！)
		  var promoProducts []Product
		  db.Model(&Product{}).Scopes(Listed, PriceBetween(100, 500)).Find(&promoProducts)
		  
		  -- 对应功能点 A
		  SELECT * FROM `users` WHERE `status` = 'active'
		  
		  -- 对应功能点 B
		  SELECT * FROM `products` WHERE `is_listed` = true
		  
		  -- 对应功能点 C
		  SELECT * FROM `products` WHERE `is_listed` = true AND `price` BETWEEN 100 AND 500
		  ```
		- 最终的代码变得极其干净、表意清晰，而且完全没有重复。如果以后“已上架”的逻辑需要修改，你只需要改动 `Listed` 那一个函数，所有使用到它的地方就全部自动更新了！这就是 `Scope` 最大的魅力。
- **Count：**
	- **作用：**帮你数一数数据库里符合条件的记录一共有多少条，但它只返回数字，不返回数据本身。
	- **基本的条件计数：**
		- 这是最常见的用法，数一数满足 `Where` 条件的记录有多少。
		- ```go
		  var pendingOrderCount int64
		  db.Model(&Order{}).Where("status = ?", "pending").Count(&pendingOrderCount)
		  
		  SELECT count(*) FROM `orders` WHERE status = 'pending'
		  ```
	- **配合 `Distinct` 使用：**
		- 有时候你不想数总数，而是想数有多少个不重复的“种类”。
		- ```go
		  var activeUserCount int64
		  db.Model(&Order{}).
		     Where("created_at > ?", today). // 限定今天
		     Distinct("user_id").             // 对 user_id 进行去重
		     Count(&activeUserCount)
		  
		  SELECT COUNT(DISTINCT(`user_id`)) FROM `orders` WHERE `created_at` > '...'
		  ```
	- **配合 `Group` 使用：**
		- `Group(...).Count(...)` 在 GORM 里的作用，不是统计每个分组里有多少条记录，而是统计“总共有多少个分组”。
		- **示例：**
			- 你想知道“今天总共有多少种不同的商品被卖出去了”。你不在乎每种商品卖了多少件，只关心卖出的商品种类数。
			- ```go
			  var distinctProductCount int64
			  // 先按 product_id 分组，然后统计有多少个组
			  db.Model(&OrderDetail{}).Where("created_at > ?", today).Group("product_id").Count(&distinctProductCount)
			  
			  SELECT count(*) FROM (SELECT `product_id` FROM `order_details` WHERE `created_at` > '...' GROUP BY `product_id`) AS `sub`
			  ```
			- GORM 先在内部执行 `GROUP BY product_id`，得到一个类似 `[商品A, 商品B, 商品C]` 的不重复商品列表，然后 `Count` 再去数这个列表里有多少项。所以，如果今天卖了100件商品，但只涉及3种不同的商品，那么 `distinctProductCount` 的结果就是 `3`。
-