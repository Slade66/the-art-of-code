- Redis 的 Sorted Set 是由唯一字符串组成的集合，每个成员都关联一个浮点数分数，并按分数升序排序。分数可以重复，但成员必须唯一。
- 常用命令
  heading:: true
	- `ZADD`
	  heading:: true
		- **语法：**`ZADD key score member [score2 member2 ...]`
		- **作用：**将一个或多个成员添加到 Sorted Set 中，并设置分数；如果成员已存在，则更新其分数。
	- `ZSCORE`
	  heading:: true
		- **语法：**`ZSCORE key member`
		- **作用：**返回指定成员的分数（score），如果成员不存在返回 nil。
	- `ZINCRBY`
	  heading:: true
		- **语法：**`ZINCRBY key increment member`
		- **作用：**将指定成员的分数增加指定值，如果成员不存在则相当于添加。
	- `ZRANGE`
	  heading:: true
		- **语法：**`ZRANGE key start stop [WITHSCORES]`
		- **作用：**返回指定区间内的成员，按分数升序排列。可以加上 `WITHSCORES` 返回对应分数。
	- `ZREVRANGE`
	  heading:: true
		- **语法：** `ZREVRANGE key start stop [WITHSCORES]`
		- **作用：** 返回指定区间内的成员，按分数降序排列。
	- `ZRANGEBYSCORE`
	  heading:: true
		- **语法：**`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
		- **作用：**返回指定分数范围内的成员，按分数升序排列。
	- `ZREVRANGEBYSCORE`
	  heading:: true
		- **语法：**`ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]`
		- **作用：**返回指定分数范围内的成员，按分数降序排列。
	- `ZREM`
	  heading:: true
		- **语法：**`ZREM key member [member2 ...]`
		- **作用：**移除一个或多个成员；如果最后一个成员被移除，集合将被删除。
	- `ZCARD`
	  heading:: true
		- **语法：**`ZCARD key`
		- **作用：**返回 Sorted Set 中成员的数量。
	- `ZRANK`
	  heading:: true
		- **语法：**`ZRANK key member`
		- **作用：**返回指定成员在 Sorted Set 中按 score 升序排列时的排名（从 0 开始）。
	- `ZREVRANK`
	  heading:: true
		- **语法：**`ZREVRANK key member`
		- **作用：**返回指定成员在 Sorted Set 中按 score 降序排列时的排名（从 0 开始）。
-