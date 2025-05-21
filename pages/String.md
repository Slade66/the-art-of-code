- 字符串类型可以存储任意格式的数据，如文本、JSON、图片，序列化后的对象。
- 单个字符串的最大长度为 512 MB。
- 常用命令
  heading:: true
	- `SET`
	  heading:: true
		- **语法**：`SET key value [NX | XX] [GET] [EX seconds | PX milliseconds]`
			- `NX`：只有 key 不存在时才设置
			- `XX`：只有 key 存在时才设置
			- `EX seconds`：设置过期时间（秒）
			- `PX milliseconds`：设置过期时间（毫秒）
			- `GET`：返回 key 原本存储的字符串值；如果 key 不存在，则返回 nil。如果 key 对应的值不是字符串类型，则返回一个错误，并且 SET 操作会被中止。
		- **作用**：
			- 将指定的 key 设置为给定的字符串值。
			- 如果该 key 已经存在，其原有的值会被覆盖，不论原值的类型为什么。
			- 在执行成功的 `SET` 操作后，key 之前设置的过期时间（TTL）也会被清除。
	- `GET`
	  heading:: true
		- **语法**：`GET key`
		- **作用**：获取指定 key 的值。若 key 不存在，返回 nil。
	- `SETEX`
	  heading:: true
		- **语法**：`SETEX key seconds value`
		- **作用**：设置 key 的值，并设置过期时间（单位：秒）。相当于 `SET` + `EX` 参数的组合。
	- `GETEX`
	  heading:: true
		- **语法**：`GETEX key [EX seconds | PX milliseconds | EXAT timestamp| PERSIST]`
		- **作用**：获取 key 的值，并可同时修改其过期时间或移除过期时间。
	- `SETNX`
	  heading:: true
		- **语法**：`SETNX key value`
		- **作用**：仅当 key 不存在时设置其值。若设置成功返回 1，否则返回 0。
	- `INCR`
	  heading:: true
		- **语法**：`INCR key`
		- **作用**：
			- 将 key 的值加 1。若 key 不存在则初始化为 0 再加 1。
			- 不会重置过期时间。
	- `DECR`
	  heading:: true
		- **语法**：`DECR key`
		- **作用**：将 key 的值减 1。若 key 不存在则初始化为 0 再减 1。
	- `MGET`
	  heading:: true
		- **语法**：`MGET key [key ...]`
		- **作用**：批量获取多个 key 的值。返回对应顺序的值列表，若 key 不存在则对应位置返回 nil。
	- `MSET`
	  heading:: true
		- **语法**：`MSET key value [key value ...]`
		- **作用**：批量设置多个 key 的值。若某些 key 已存在将被覆盖。
-