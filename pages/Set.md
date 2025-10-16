- Redis 的 Set 是一种无序的、不重复的字符串集合。
- 常用命令
  heading:: true
	- `SADD`
	  heading:: true
		- **语法**：`SADD key member [member ...]`
		- **作用**：向集合中添加一个或多个元素。集合不存在会自动创建。
	- `SREM`
	  heading:: true
		- **语法**：`SREM key member [member ...]`
		- **作用**：从集合中移除一个或多个元素。
	- `SISMEMBER`
	  heading:: true
		- **语法**：`SISMEMBER key member`
		- **作用**：判断元素是否存在于集合中。`1` 为存在，`0` 为不存在。
	- `SMEMBERS`
	  heading:: true
		- **语法**：`SMEMBERS key`
		- **作用**：返回集合中的所有元素。
	- `SCARD`
	  heading:: true
		- **语法**：`SCARD key`
		- **作用**：返回集合中元素的数量。
	- `SUNION`
	  heading:: true
		- **语法**：`SUNION key1 key2 [key3 ...]`
		- **作用**：返回多个集合的并集（所有元素，不重复）。
	- `SINTER`
	  heading:: true
		- **语法**：`SINTER key1 key2 [key3 ...]`
		- **作用**：返回多个集合的交集（共同存在的元素）。
	- `SDIFF`
	  heading:: true
		- **语法：**`SDIFF key1 key2 [key3 ...]`
		- **作用：**从第一个集合中剔除其它集合中也出现的元素，返回其独有部分。
-