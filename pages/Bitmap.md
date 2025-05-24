- Bitmap（位图）本质上是一个二进制位数组，用 0 和 1 来表示状态。它通过二进制位高效地存储和处理大量布尔信息，既节省空间，又支持高效的位操作。
- 常用命令
  heading:: true
	- `SETBIT`
	  heading:: true
		- **语法：**`SETBIT key offset value`
		- **作用：**将指定 key 的第 offset 位设置为 value（0 或 1），如果 key 不存在会自动创建。
	- `GETBIT`
	  heading:: true
		- **语法：**`GETBIT key offset`
		- **作用：**获取指定 key 的第 offset 位的值（0 或 1）。
	- `BITCOUNT`
	  heading:: true
		- **语法：**`BITCOUNT key [start end]`
		- **作用：**统计 key 中 bit=1 的个数。可指定起止字节范围（按字节）。
	- `BITOP`
	  heading:: true
		- **语法：**`BITOP operation destkey key1 [key2 ...]`
		- **作用：**对一个或多个 Bitmap 做按位运算（`AND`、`OR`、`XOR`、`NOT`），结果保存到 `destkey`。