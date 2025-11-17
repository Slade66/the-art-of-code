-
- ## 怎么用？
	- 开头引用：`import "validate/validate.proto";`
	  logseq.order-list-type:: number
	- 在字段上用：`[(validate.rules).string = { ... }]`
	  logseq.order-list-type:: number
	- ### 字符串
		- **`const`**
			- **含义**：字符串必须“恰好等于”这个值。
			- 示例：`const: "admin"`。
		- **`len`**
			- **含义**：必须正好是指定数量的“字符”。
			- 注意：按 rune 数，不是按字节数；一个中文是 1 个字符但多字节。
		- **`min_len` / `max_len`**
			- **含义**：最小/最大字符数（也是按 rune 数）。
			- 示例：`string name = 1 [(validate.rules).string = {min_len: 1, max_len: 20}];`
			- **坑点**：`"   "`（三个空格）长度是 3，**可以通过 `min_len = 1`**，不会帮你 TrimSpace。
		- **`len_bytes` / `min_bytes` / `max_bytes`**
			- **含义**：按“字节长度”约束，而不是字符数。
		- **`pattern`**
			- **含义**：必须匹配给定正则（RE2 语法）。
		- **`prefix` / `suffix`**
			- **prefix**：必须以给定子串开头。
			- **suffix**：必须以给定子串结尾。
		- **`contains` / `not_contains`**
			- **contains**：字符串中必须包含某子串。
			- **not_contains**：字符串中不能包含某子串。
		- **`in` / `not_in`**
			- **in**：值必须是列表中的一个。
			- **not_in**：值不能是列表中的任何一个。
		- 同一时间只能选其中一个（`oneof well_known`）：
			- **`email`**
				- 必须是合法邮箱（RFC 5322）。
			- **`hostname`**
				- 合法域名（RFC 1034，不支持 IDN）。
			- **`ip` / `ipv4` / `ipv6`**
				- `ip`：合法 IPv4 或 IPv6。
				- `ipv4`：只允许 IPv4。
				- `ipv6`：只允许 IPv6（不带 `[]`）。
			- **`uri` / `uri_ref`**
				- `uri`：必须是**绝对 URI**（RFC 3986），如 `https://example.com/path`。
				- `uri_ref`：可以是相对或绝对 URI。
			- **`address`**
				- 要么是合法 `hostname`，要么是合法 IP 地址。
			- **`uuid`**
				- 必须是合法 UUID（RFC 4122）。
		- `ignore_empty`
			- **含义**：如果字段是“空字符串”，就**跳过所有其他规则**，不报错。
			- 典型用法：某字段是可选的，但一旦非空，就要满足某规则：
				- ```proto
				  string nickname = 1 [
				    (validate.rules).string = {
				      max_len: 20
				      ignore_empty: true
				    }
				  ];
				  ```
-