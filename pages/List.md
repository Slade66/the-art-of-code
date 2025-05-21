是什么？
heading:: true
	- 列表是一种按插入顺序排序的字符串集合，底层实现为双向链表，允许重复元素。
	- 可在头部或尾部执行插入、弹出和读取等操作。
- 常用命令
  heading:: true
	- `LPUSH`
	  heading:: true
		- **语法**：`LPUSH key value [value ...]`
		- **作用**：从列表左侧插入一个或多个元素。如果 key 不存在，会创建一个新列表。新元素会被添加到列表头部（最左边）。
	- `RPUSH`
	  heading:: true
		- **语法**：`RPUSH key value [value ...]`
		- **作用**：从列表右侧插入一个或多个元素。如果 key 不存在，会创建一个新列表。新元素会被添加到列表尾部（最右边）。
	- `LPOP`
	  heading:: true
		- **语法**：`LPOP key [count]`
		- **作用**：
			- 默认情况下，该命令会从列表的开头（列表最左边）弹出（移除并返回）一个元素。若提供了可选参数 `count`，则返回最多 `count` 个元素，具体数量取决于列表的长度。
			- 如果列表为空或 key 不存在，返回 nil。
	- `RPOP`
	  heading:: true
		- **语法**：`RPOP key [count]`
		- **作用**：移除并返回列表最右边的元素。同 `LPOP`。
	- `LRANGE`
	  heading:: true
		- **语法**：`LRANGE key start stop`
		- **作用**：返回列表中从指定索引范围的元素子列表，包括 `start` 和 `stop` 位置的元素。索引从 0 开始，0 表示第一个元素，负数表示从尾部开始计数（如 -1 表示最后一个元素）。
	- `LLEN`
	  heading:: true
		- **语法**：`LLEN key`
		- **作用**：返回列表的长度（元素个数）。如果 key 不存在，返回 0。
	- `BLPOP`
	  heading:: true
		- `BLPOP` 是 `LPOP` 的阻塞版本。
		- 当指定的列表都为空时，它会阻塞连接，直到有元素可弹出或超时。
		- **语法**：`BLPOP key [key ...] timeout`
		- **作用**：按给定的 key 顺序依次检查，返回第一个非空列表中最左侧的元素，并将其移除。如果所有列表为空，则阻塞等待最多 `timeout` 秒（0 表示无限阻塞），直到有元素可用；若超时仍无元素，返回 nil。
	- `BRPOP`
	  heading:: true
		- **语法**：`BRPOP key [key ...] timeout`
		- **作用**：从右侧弹出元素。行为与 `BLPOP` 类似，只是从右侧操作。
-