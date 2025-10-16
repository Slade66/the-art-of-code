- Redis 的 Hash（哈希）是一种键值对集合。它类似于 Java 中的 `Map<String, String>`，它存储在 Redis 的一个键下面。
- 常用命令
  heading:: true
	- `HSET`
	  heading:: true
		- **语法**：`HSET key field value [field2 value2 ...]`
		- **作用**：设置一个或多个字段及其值，如果字段存在则会覆盖旧值。
	- `HGET`
	  heading:: true
		- **语法**：`HGET key field`
		- **作用**：获取某个字段的值。
	- `HGETALL`
	  heading:: true
		- **语法：**`HGETALL key`
		- **作用**：获取该 Hash 中所有字段及其对应的值。
	- `HDEL`
	  heading:: true
		- **语法：**`HDEL key field [field2 ...]`
		- **作用：**删除一个或多个字段及其对应的值。如果所有字段被删除，该 Hash 也将被移除。
	- `HEXISTS`
	  heading:: true
		- **语法：**`HEXISTS key field`
		- **作用：**检查 Hash 中是否存在指定字段，存在返回 `1`，否则返回 `0`。
	- `HLEN`
	  heading:: true
		- **语法：**`HLEN key`
		- **作用：**返回 Hash 中字段的数量。
	- `HMGET`
	  heading:: true
		- **语法：**`HMGET key field1 field2 ...`
		- **作用：**获取多个字段的值，按请求顺序返回，每个字段不存在时返回 `nil`。
	- `HKEYS`
	  heading:: true
		- **语法：**`HKEYS key`
		- **作用：**获取 Hash 中所有字段（field）的列表。
	- `HVALS`
	  heading:: true
		- **语法：**`HVALS key`
		- **作用：**获取 Hash 中所有字段对应的值列表。
-