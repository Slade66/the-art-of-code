-
- ### 有什么用？
	- **作用：**`HAVING` 用于对 `GROUP BY` 分组后的结果进行筛选。
	- **先筛选，再分组，再筛选：**先用 `WHERE` 在分组前过滤掉不需要的行，再用 `GROUP BY` 对数据分组，最后用 `HAVING` 去除不需要的分组。
- ### `HAVING` vs `WHERE`
	- **作用对象：**`WHERE` 针对表中的每一行记录，`HAVING` 针对 `GROUP BY` 分组后的结果。
	- **执行时机：**`WHERE` 在分组之前执行，`HAVING` 在分组之后执行。
	- **能否使用聚合函数：**`WHERE` 不能，`HAVING` 可以。
- ### 怎么用？
	- `HAVING` 总是跟在 `GROUP BY` 后面。
	- `HAVING` 总是与聚合函数一起使用，因为聚合函数的计算结果是 `HAVING` 子句进行判断的依据。
- ### 聚合函数与 `HAVING`
	- 聚合函数既可以写在 `HAVING` 子句中，也可以写在 `SELECT` 列表中并通过别名引用。
	- #### 方式一：在  `HAVING`  子句中直接写聚合函数
		- 这是最符合 SQL 标准的写法，几乎所有数据库系统都支持。
		- `HAVING` 子句可以直接调用聚合函数，对分组后的行进行聚合计算并筛选。
		- **示例**：找出购买次数超过 1 次的客户。
			- ```sql
			  SELECT
			      customer_id,
			      COUNT(*) -- 用于展示每个客户的订单总数
			  FROM
			      orders
			  GROUP BY
			      customer_id
			  HAVING
			      COUNT(*) > 1; -- 用于筛选订单总数大于 1 的客户
			  ```
			- 在这个例子里，`COUNT(*)` 出现了两次：
				- 在 `SELECT` 中用于展示每个客户的订单总数。
				- 在 `HAVING` 中用于筛选订单总数大于 1 的客户。
	- #### 方式二：在 `SELECT` 中定义别名，并在 `HAVING` 中引用
		- MySQL 支持一种更清晰、更易于维护的写法：先在 `SELECT` 列表中为聚合函数结果起一个别名，然后在 `HAVING` 子句中直接引用该别名。
		- **示例**：同样是找出购买次数超过 1 次的客户。
			- ```sql
			  SELECT
			      customer_id,
			      COUNT(*) AS total_orders -- 在 SELECT 中计算聚合结果并定义别名
			  FROM
			      orders
			  GROUP BY
			      customer_id
			  HAVING
			      total_orders > 1; -- 在 HAVING 中直接使用别名
			  ```
	- 在写法上，你可以直接在 `HAVING` 中重复写一遍聚合函数。但更优的做法是在 `SELECT` 列表中为聚合函数的结果起一个清晰的别名，然后在 `HAVING` 中引用这个别名。这样做有两个好处：
		- **可读性**：`HAVING total_orders > 1` 比 `HAVING COUNT(*) > 1` 更清晰，业务含义一目了然。
		- **可维护性**：当聚合逻辑变复杂时，只需在 `SELECT` 中修改一次，避免了 `SELECT` 和 `HAVING` 同时修改可能带来的遗漏。”
-