- **标准且完整的 MySQL 建表语句：**
	- ```sql
	  CREATE TABLE IF NOT EXISTS `t_orders` (
	    -- 1. 主键：使用 BIGINT UNSIGNED，自增，且无符号（不允许负数，扩大范围）
	    `id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键ID',
	  
	    -- 2. 字符串与唯一性：订单号通常是业务生成的唯一字符串
	    `order_no` VARCHAR(64) NOT NULL COMMENT '业务订单号，全局唯一',
	  
	    -- 3. 外键关联字段：通常与关联表主键类型一致，加索引
	    `user_id` BIGINT UNSIGNED NOT NULL COMMENT '用户ID',
	    
	    -- 4. 金额字段：严禁使用 FLOAT/DOUBLE，必须用 DECIMAL(M, D) 保证精度
	    `total_amount` DECIMAL(10, 2) NOT NULL DEFAULT '0.00' COMMENT '订单总金额(元)',
	    
	    -- 5. 状态字段：使用 TINYINT 节省空间，配合枚举值
	    `order_status` TINYINT NOT NULL DEFAULT 0 COMMENT '订单状态: 0-待支付, 1-已支付, 2-已发货, 3-已完成, 4-已取消',
	  
	    -- 6. 布尔类型：MySQL 中 BOOL 等同于 TINYINT(1)，常用于逻辑删除
	    `is_deleted` TINYINT(1) NOT NULL DEFAULT 0 COMMENT '逻辑删除: 0-未删除, 1-已删除',
	  
	    -- 7. 文本存储：使用 TEXT 存储较长文本，如备注
	    `remark` VARCHAR(255) NOT NULL DEFAULT '' COMMENT '用户备注',
	  
	    -- 8. JSON 类型 (MySQL 5.7+)：存储非结构化数据，如快照信息或扩展属性
	    `extra_info` JSON COMMENT '扩展信息(JSON格式): {"device": "ios", "version": "1.0"}',
	  
	    -- 9. 时间字段：自动初始化和自动更新
	    `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
	    `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
	    
	    -- 10. 日期字段：仅存储日期
	    `delivery_date` DATE DEFAULT NULL COMMENT '预计发货日期',
	  
	    -- === 约束区域 (Constraints) ===
	    
	    -- 主键约束
	    PRIMARY KEY (`id`),
	  
	    -- 唯一约束：保证订单号不重复
	    UNIQUE KEY `uk_order_no` (`order_no`),
	  
	    -- === 索引区域 (Indexes) ===
	    
	    -- 普通索引：用于查询某个用户的订单
	    KEY `idx_user_id` (`user_id`),
	  
	    -- 复合索引：用于查询"最近下单"或"某状态下的订单"，注意最左前缀原则
	    KEY `idx_status_created` (`order_status`, `created_at`),
	  
	    -- 全文索引 (可选)：如果需要对备注进行搜索 (需 InnoDB 引擎)
	    FULLTEXT KEY `ft_remark` (`remark`),
	  
	    -- Check 约束 (MySQL 8.0.16+)：确保数据逻辑正确
	    CONSTRAINT `chk_amount_positive` CHECK (`total_amount` >= 0)
	  
	  ) 
	  -- === 表选项 (Table Options) ===
	  ENGINE = InnoDB                            -- 存储引擎：默认 InnoDB，支持事务
	  AUTO_INCREMENT = 1                         -- 自增起始值
	  DEFAULT CHARSET = utf8mb4                  -- 字符集：推荐 utf8mb4 (支持Emoji)
	  COLLATE = utf8mb4_0900_ai_ci               -- 排序规则：8.0 默认，不区分大小写，准确性高
	  ROW_FORMAT = DYNAMIC                       -- 行格式：优化变长字段存储
	  COMMENT = '用户订单表';                      -- 表注释
	  ```
	- **`CHECK`**：MySQL 8.0 引入的新特性，可以像编程一样在数据库层面校验数据（例如：金额不能小于 0）。
- **引擎：**
	- **`InnoDB`：**必须选它。支持事务（ACID）、行级锁（Row-level Locking）和外键。
- **外键行为：**
	- 在互联网高并发场景下，通常禁止使用物理外键。因为外键检查会降低写入性能并可能导致死锁。数据的一致性通常由应用层代码（Java/Python/Go）逻辑来保证。
- **字符集：**
	- 字符集的选择直接影响数据的存储空间和多语言支持能力。选择正确的字符集是迈向全球化的基石。
	- **utf8mb3 vs utf8mb4**
		- MySQL 中的 `utf8` 实际上是指 `utf8mb3`，它最多使用 3 个字节存储一个字符。这已经能覆盖世界上绝大多数常用字符，但无法容纳补充字符。典型例子是 Emoji（如 🐬）以及部分生僻汉字，这些字符需要 4 个字节才能表示。
		- `utf8mb4` 是真正的 UTF-8 实现，支持 4 字节字符，世界各地的文字都能正确表示，Emoji 不会变成乱码，是现代 Web 开发里通用、默认、最被广泛接受的编码标准。对于 ASCII 字符（如英文、数字），`utf8mb4` 和 `utf8mb3` 一样只占用 1 个字节。
		- **结论：**`utf8`（实为 `utf8mb3`）已不再适应现代需求，请使用 `utf8mb4`。
- **排序规则：**
	- 排序规则决定了字符的比较和排序方式。
	- `utf8mb4_general_ci`：较早期的默认值，比较时做了简化处理（例如把某些不同字符视作相同）。速度快，但准确性一般。
	- `utf8mb4_unicode_ci`：基于 Unicode 标准，排序更精确（例如能正确区分和处理德语 ß 等字符），但性能稍逊。
	- `utf8mb4_0900_ai_ci`：MySQL 8.0 的默认值，基于 Unicode 9.0。`ai` 表示不区分重音，`ci` 表示不区分大小写。利用了新的 CPU 指令优化，兼具高性能与高准确度，是目前建表时的最佳选择。
	- **结论：**所有新表应强制指定 `CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci`。
- ## 关系
	- **一对一关系：**
		- 通常用于将大表拆分或隔离敏感数据。
		- 实现方式是在任意一方（通常是访问频率较低的一方）添加外键指向另一方的主键，并对该外键设置唯一约束（UNIQUE Constraint）。
	- **一对多关系：**
		- 外键放置在多的一方。
			- 例如，一个部门（One）有多个员工（Many），则在"员工"表中添加"部门ID"作为外键。
	- **多对多关系：**
		- MySQL 不支持直接的多对多存储，必须引入关联表（Junction Table / Association Table）来分解关系。
		- 例如，"学生"和"课程"是多对多关系。
			- 我们需要创建一个名为 `student_courses` 的中间表：
				- ```sql
				  CREATE TABLE student_courses (
				      student_id INT NOT NULL,
				      course_id INT NOT NULL,
				      enrollment_date DATETIME DEFAULT CURRENT_TIMESTAMP,
				      PRIMARY KEY (student_id, course_id),
				      FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
				      FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
				  ) ENGINE=InnoDB;
				  ```
			- 在此模型中，`student_courses` 表不仅解耦了多对多关系，还承载了关系本身的属性（如 `enrollment_date`）。
			- **关键索引策略：**在关联表中，联合主键 `(student_id, course_id)` 自然地为基于 `student_id` 的查询提供了索引。然而，为了高效地查询"某课程的所有学生"，必须为 `course_id` 单独创建索引或创建反向的联合索引 `(course_id, student_id)`。这是新手常犯的错误，导致反向查询全表扫描。
- ## 数据类型
	- **整数类型：**
		- | 类型      | 字节数 | 有符号范围              | 无符号 (UNSIGNED) 范围      |
		  |-----------|--------|--------------------------|------------------------------|
		  | TINYINT   | 1      | -128 ~ 127               | 0 ~ 255                      |
		  | SMALLINT  | 2      | -32,768 ~ 32,767         | 0 ~ 65,535                   |
		  | MEDIUMINT | 3      | -8.3M ~ 8.3M             | 0 ~ 16.7M                    |
		  | INT       | 4      | -21亿 ~ 21亿             | 0 ~ 42亿                     |
		  | BIGINT    | 8      | -9.22E18 ~ 9.22E18       | 0 ~ 1.84E19                  |
	- **定点数与浮点数：**
		- | 类型            | 特点                                         | 说明与示例                                                       | 适用场景                          |
		  |-----------------|----------------------------------------------|------------------------------------------------------------------|-----------------------------------|
		  | DECIMAL(M, D)   | 存储精确值                                   | 以字符串形式编码；如 DECIMAL(10,2) 可表示 99999999.99           | 金融、计费，任何需要绝对精度的场景 |
		  | FLOAT / DOUBLE  | 存储近似值，计算速度快                       | 可能出现精度误差，如 0.1 + 0.2 ≠ 0.3                            | 科学计算、对精度不敏感的业务       |
	- **字符串：**
		- **`CHAR(N)`：**定长字符串。如果内容长度固定（如 MD5 哈希、国家代码），`CHAR` 性能更好，且不容易产生内存碎片。
		- **`VARCHAR(N)`：**它的全称是 Variable Character（可变长字符），用于存储长度可变的字符串。
		  collapsed:: true
			- 相比于固定长度的 `CHAR` 类型，对于长度波动大的数据（如名字、地址），它比 `CHAR` 更节省空间。
			- `VARCHAR(N)` 中的 **`N`** 代表的是**字符数**，而不是字节数。
			- `VARCHAR` 在磁盘上的实际存储方式包含两个部分：
			  collapsed:: true
				- **真实数据**：实际存储的字符串内容。
				- **长度前缀**：用于记录该数据到底占用了多少字节。
					- 如果列的值占用的字节数 **小于 255**，使用 **1 个字节** 来记录长度。
					- 如果列的值占用的字节数 **大于 255**，使用 **2 个字节** 来记录长度。
			- **VARCHAR(N) 中 N 最大能是多少？**
			  collapsed:: true
				- MySQL 有一个硬性限制：一行记录（Row）的最大长度不能超过 65,535 字节。
				- 首先要减去 NULL 标识位和长度前缀。
				- 如果你的表中还有其他列（如 `INT`, `DATETIME`），那么留给 `VARCHAR` 的空间会更小。
				- **影响 N 大小的关键因素：字符集**
					- 不同的字符集，每个字符占用的字节数不同，这直接决定了 `N` 的上限。
					- | **字符集** | **单个字符最大字节** | **VARCHAR(N) 的最大 N (近似值)** | **说明**|
					  | ---- | ---- | ---- |
					  | **latin1** | 1 byte | ~65,532 | 除去长度前缀和少量开销 |
					  | **gbk** | 2 bytes | ~32,766 | (65535-2) / 2 |
					  | **utf8** | 3 bytes | ~21,844 | (65535-2) / 3 |
					  | **utf8mb4** | 4 bytes | **~16,383** | (65535-2) / 4，目前最常用的字符集 |
			- **内存消耗：**
				- 即使你在磁盘上存的很省，MySQL 引擎在将数据读入内存进行操作（如排序、临时表）时，往往会按 `VARCHAR(N)` 定义的宽度分配内存。
				- **例子：**
				  collapsed:: true
					- 假设我们有两张表存同样的数据 "Alice" (5字节)，但表定义不同：
						- **表 A：**`name VARCHAR(10)`
						- **表 B：**`name VARCHAR(100)`
					- 当执行 `SELECT * FROM table ORDER BY name` 时，内存分配对比如下：
						- ```
						  假设：字符集为 utf8mb4 (每字符最大 4 字节)
						  数据："Alice"
						  
						  【表 A: VARCHAR(10)】
						  需要的内存槽位 = 10 * 4 = 40 字节
						  内存布局：
						  +------------------------------------------+
						  | Alice . . . . . . . . . . . . . . . . . .|  <- 仅仅浪费少量填充空间
						  +------------------------------------------+
						  | 实际数据 |           空闲/填充             |
						  
						  -----------------------------------------------------------------------
						  
						  【表 B: VARCHAR(100)】
						  需要的内存槽位 = 100 * 4 = 400 字节
						  内存布局：
						  +-----------------------------------------------------------------------+
						  | Alice . . . . . . . . . . (此处省略 300 多字节的空白) . . . . . . . . . | <- 巨大的内存浪费！
						  +-----------------------------------------------------------------------+
						  | 实际 |                                                                |
						  | 数据 |                         巨大的无效填充区                         |
						  |      |                                                                |
						  +-----------------------------------------------------------------------+
						  ```
				- **坏习惯：**`VARCHAR(255)` 一把梭，或者为了省事直接定义 `VARCHAR(5000)`。
				- **后果：**
					- 如果只需要存 10 个字符，却定义了 `VARCHAR(1000)`，在排序时，MySQL 可能会分配更多的内存缓冲区，导致性能下降。
					- **内存利用率低：** 如果你定义了 `VARCHAR(5000)` 但只存了几个字，排序时每一个行记录在内存中都会占用 20KB (5000*4) 的预留空间。
					- **磁盘临时表：** 因为单行数据在内存中“虚胖”，导致 `sort_buffer` 能容纳的行数变少。一旦内存不够，MySQL 就不得不把数据交换到磁盘上进行排序，导致性能急剧下降。
				- **精准定义 N：**N 够用就好，不要过大。虽然 `VARCHAR(10)` 和 `VARCHAR(100)` 存 "abc" 在磁盘上占用空间一样，但在**内存处理**时，`VARCHAR(100)` 消耗更多。请根据业务实际需求设定 N。
			- **索引长度限制：**
				- InnoDB 引擎对索引键的长度有限制（通常是 3072 字节）。
				- 在 `utf8mb4` 下，`VARCHAR(1000)` 是无法建立全字段索引的（1000 * 4 = 4000 > 3072）。
				- **解决方案：**使用前缀索引。例如 `INDEX(my_col(60))`，只索引前 60 个字符。
			- **Update 导致的页分裂：**
				- 如果一个 `VARCHAR` 列原本存储了 "abc"（很短），后来 Update 成了 "abc...xyz"（很长），当前数据页（Page）可能没有足够的空间容纳增长后的行。
				- InnoDB 就必须进行页分裂（Page Split），将部分数据移动到新页。这会增加磁盘 I/O 并导致索引碎片。
		- **`TEXT` 系列：**`TINYTEXT`, `TEXT`, `MEDIUMTEXT`, `LONGTEXT`。用于存储大文本。关键区别在于 `TEXT` 列通常存储在页外（Off-page storage），主索引中只保留指针。
			- 查询时应避免 `SELECT *` 包含不必要的 `TEXT` 列，以免引发大量的磁盘随机 I/O。
		- **VARCHAR vs CHAR**
			- ...
	- **日期时间：**
		- **`DATETIME`：**用于存储日期和时间。
		  collapsed:: true
			- **显示格式：** `YYYY-MM-DD HH:MM:SS` (例如：`2023-10-27 14:30:05`)
			- **支持范围：** `1000-01-01 00:00:00` 到 `9999-12-31 23:59:59`。
			- **存储空间：**
			  collapsed:: true
				- 占用 5 个字节（MySQL 5.6.4 之前是 8 个字节）。
				- 如果需要存储毫秒/微秒，需要额外的空间：
					- `DATETIME(0)`: 精确到秒 (5 字节)。
					- `DATETIME(3)`: 精确到毫秒 (5 + 2 = 7 字节)。
					- `DATETIME(6)`: 精确到微秒 (5 + 3 = 8 字节)。
			- **不进行时区转换：**
			  collapsed:: true
				- 它不包含时区信息，存入什么，取出来就是什么。
				- 当你存入 `2023-10-01 08:00:00`，无论你修改了 MySQL 服务器的时区，还是客户端连接的时区，读取出来的值永远是 `2023-10-01 08:00:00`。
			- **自动初始化与更新：**
			  collapsed:: true
				- 从 MySQL 5.6.5 开始，`DATETIME` 也支持自动初始化和更新（以前只有 `TIMESTAMP` 支持）。
				- ```sql
				  CREATE TABLE orders (
				      id INT AUTO_INCREMENT PRIMARY KEY,
				      order_status VARCHAR(50),
				      
				      -- 创建时自动填入当前时间
				      created_at DATETIME DEFAULT CURRENT_TIMESTAMP, 
				      
				      -- 创建时自动填入，且每次更新该行数据时，自动更新为当前时间
				      updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
				  );
				  ```
		- **`TIMESTAMP`：**存储自 1970-01-01 UTC 以来的秒数（4 字节）。它具有时区感知能力：存入时由当前时区转为 UTC，取出时转回当前时区。
		  collapsed:: true
			- **2038 年问题：**标准 `TIMESTAMP` 将在 2038 年溢出。对于长期业务数据，推荐使用 `DATETIME` 或 `BIGINT`（存储毫秒时间戳）以规避此风险。
		- **DATETIME vs TIMESTAMP**
		  collapsed:: true
			- | **特性** | **DATETIME** | **TIMESTAMP** |
			  | ---- | ---- | ---- |
			  | **存储范围** | `1000` 年 - `9999` 年 | `1970` 年 - `2038` 年 (UTC 范围) |
			  | **存储大小** | 5 字节 (+小数秒空间) | 4 字节 (+小数秒空间) |
			  | **时区处理** | **无转换** (绝对值) | **自动转换** (存入转UTC，取出转当前时区) |
			  | **性能** | 极快 (直接存取) | 稍慢 (涉及转换) |
			  | **主要用途** | 生日、未来日程、固定时间点 | 记录数据变更时间、审计日志 |
			  | **2038年问题** | 无 | 有 (部分旧版本受限) |
	- **JSON：**
		- **二进制存储：**JSON 数据不是作为文本字符串存储，而是被解析为优化的二进制格式（JDOM）。这使得读取子节点（如 `doc->"$.key"`）时无需解析整个文档，性能极高。
		- 允许你在关系型数据库中存储 NoSQL 风格的数据，不再需要把一堆字段硬塞进表中。
- ## 命名规范
	- **Snake Case：**推荐使用下划线分隔的小写字母（如 `user_orders`），避免使用 CamelCase。
	- **复数 vs 单数：**在 ORM 映射中，表名通常建议使用复数（`users`），代表记录的集合；而类名使用单数（`User`）。
	- **索引的名字应该能看出用途：**普通索引用 `idx_列名`，唯一索引用 `uq_列名`，外键则采用 `fk_子表_父表` 这样的前缀，方便识别和维护。
	- **反引号：**表名和字段名都使用了 ` ` 包裹，防止字段名（如 `order`、`status`）与 MySQL 保留字冲突。
	- **前缀：**表名使用 `t_` 前缀（`t_orders`），这是很多公司的规范，用于区分业务表和视图等。
- ## DESC
	- `Key` 的含义：
		- `PRI`：主键
		- `UNI`：唯一索引
		- `MUL`：普通索引/非唯一索引（允许多行有相同值，所以叫 MUL = multiple）
-