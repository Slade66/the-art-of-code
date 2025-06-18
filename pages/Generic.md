key 的增删改查
heading:: true
	- `DEL`
	  heading:: true
		- **语法**：`DEL key [key ...]`
		- **作用**：删除指定的 1 个或多个 key。返回删除掉的 key 数量。
	- `FLUSHDB`
	  heading:: true
		- **作用：**删除当前选择的数据库中的所有 key。
	- `FLUSHALL`
	  heading:: true
		- **作用：**删除所有数据库中的所有 key。
	- `EXISTS`
	  heading:: true
		- **语法**：`EXISTS key [key ...]`
		- **作用**：检查给定的 key 是否存在。返回存在的 key 数量，不存在返回 0。
	- `TYPE`
	  heading:: true
		- **语法**：`TYPE key`
		- **作用**：返回 key 对应的 value 的数据类型。
	- `RENAME`
	  heading:: true
		- **语法**：`RENAME key newkey`
		- **作用**：将 key 重命名为 newkey。如果 newkey 已存在，则会被覆盖。
	- `keys`
	  heading:: true
		- **语法**：`KEYS pattern`
		- **作用**：查找所有匹配给定 pattern 的 key。
			- pattern 是一种字符串匹配规则，用于在 Redis 命令中根据特定格式筛选符合条件的 key。
			- 常见的 pattern 通配符：
				- **`*`**：匹配任意数量的字符（包括 0 个）
				  id:: 682d9dbf-4253-494f-a585-180aac3055c0
				- `?`：匹配任意单个字符
				- `[]`：匹配括号内任意一个字符
		- **注意**：`KEYS` 命令会遍历整个 Redis 数据库，在数据量大时性能开销很高，不推荐用于生产环境。
- key 的过期时间管理
  heading:: true
	- `EXPIRE`
	  heading:: true
		- **语法**：`EXPIRE key seconds`
		- **作用**：为指定的 key 设置过期时间（以秒为单位），到时间后 key 会被自动删除。支持更新已有的过期时间。如果 key 不存在，返回 0；设置成功返回 1。
	- `TTL`
	  heading:: true
		- **语法**：`TTL key`
		- **作用**：查看指定 key 距离过期还剩多少秒。
			- 返回值：
				- 正整数：剩余秒数
				- `-1`：key 存在但没有设置过期时间
				- `-2`：key 不存在
	- `PERSIST`
	  heading:: true
		- **语法**：`PERSIST key`
		- **作用**：移除指定 key 的过期时间，使其变为永久存在。
			- 如果 key 存在且原本有过期时间，返回 1；否则返回 0。
-