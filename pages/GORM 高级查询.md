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
	- **用途：**GORM 允许你把一个查询的结果当作另一个查询的条件。
	- **比喻：**老师想表扬“班级平均分以上的同学”。他需要两步：1. 先算出“班级平均分”是多少（比如85分）。2. 再找出所有分数高于85分的同学。
	- ```go
	  // 找出所有价格高于平均价格的商品
	  avgPriceQuery := db.Model(&Product{}).Select("AVG(price)")
	  db.Where("price > (?)", avgPriceQuery).Find(&products)
	  
	  // SELECT * FROM `products` WHERE price > (SELECT AVG(price) FROM `products`)
	  ```
	- **FROM 子查询：**
		- `FROM` 子查询允许你将一个查询的结果作为虚拟表来使用，并在外层查询中引用它。
		- ```go
		  // 创建一个子查询，计算平均价格
		  subQuery := db.Model(&Product{}).Select("AVG(price) AS avg_price")
		  
		  // 使用 FROM 子查询将计算出的平均价格作为一个临时表
		  var products []Product
		  db.Table("(?) AS avg_price_table", subQuery).Where("price > avg_price_table.avg_price").Find(&products)
		  
		  // SELECT * FROM `products` 
		  // WHERE price > (
		  //    SELECT AVG(price) AS avg_price FROM `products`
		  // )
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
-