有什么用？
heading:: true
	- 公共表表达式（Common Table Expression，简称 CTE）是在当前 SQL 查询中临时存在的、有名字的结果集，作为后续语句的数据源。
	- **使用公用表表达式（CTE）主要有以下好处**：
		- **提升可读性与可维护性**：CTE 允许将复杂的查询分解成独立的、具有描述性名称的逻辑单元。这有助于将深层嵌套的子查询结构扁平化，使得主查询逻辑更清晰，整个 SQL 语句更易于阅读和维护。
		- **实现逻辑复用**：如果某段查询逻辑（例如一个子查询或计算步骤）需要在后续查询中被多次引用，只需定义一次 CTE，便可通过其名称在主查询或其他 CTE 中重复使用，避免了代码冗余。
- 怎么用？
  heading:: true
	- **作用范围**：仅限于紧跟其定义的那条 SQL 语句。该语句执行结束后，CTE 便不再有效。
	- **定义 CTE**：
		- **`WITH` 子句**：
			- 以 `WITH` 关键字开头。
			- 随后定义一个或多个 CTE。每个 CTE 的格式为 `cte_name AS (SELECT ...)`：
				- `cte_name` 是你为这个临时的、命名的结果集指定的名称。
				- 括号内的 `SELECT` 语句定义了该 CTE 的内容。结尾不要加 `;`。
			- 如果定义了多个 CTE，它们之间需用逗号 `,` 分隔。需要注意的是，后定义的 CTE 可以引用在它之前定义的 CTE。
		- **主查询**：
			- 紧跟在 `WITH` 子句（包含所有 CTE 定义）之后的是一个单独的主 SQL 语句（通常是 `SELECT`，也可以是 `INSERT`, `UPDATE`, `DELETE`）。
			- 该主查询可以像引用普通表一样，通过 `cte_name` 来引用 `WITH` 子句中定义的 CTE。
			- 只能写在 `FROM` 或 `JOIN` 子句后。
	- ```sql
	  WITH
	    cte_name1 AS (
	      SELECT column1, column2 FROM tableA WHERE condition1
	    ), -- 第一个 CTE 定义结束，用逗号分隔
	    cte_name2 AS (
	      -- 第二个 CTE 可以引用第一个 CTE (cte_name1)
	      SELECT t1.column1, t2.column3
	      FROM cte_name1 t1
	      JOIN tableB t2 ON t1.column2 = t2.id
	      WHERE condition2
	    )
	  -- 主查询开始，可以使用前面定义的 cte_name1 和 cte_name2
	  SELECT cte1.column1, cte2.column3
	  FROM cte_name1 cte1
	  JOIN cte_name2 cte2 ON cte1.column1 = cte2.column1;
	  ```