概念
heading:: true
	- 简称 Protobuf，是 Google 开发的一种与语言无关、平台无关、具有良好可扩展性的序列化数据结构机制。
	- Protobuf 是用于定义数据结构并对数据进行序列化和反序列化的工具，相较于 JSON/XML，它是一种更高效的数据通信格式，用于在不同服务之间传输结构化数据（通信）。
- `.proto` 文件
  heading:: true
	- 使用 `.proto` 文件定义数据结构。
	- **例子：**
		- ```protobuf
		  syntax = "proto3";  // 指定语法版本，proto3 是推荐的
		  
		  // 定义一个消息类型（类似类或结构体）
		  message Person {
		    string name = 1;    // 字段类型 字段名 = 字段编号
		    int32 id = 2;
		    string email = 3;
		    repeated string phones = 4; // 数组（列表）
		  }
		  ```
	- **字段的类型：**
		- `int32`，32 位整数
		- `int64`，64 位整数
		- `string`，字符串
		- `bool`，布尔值
		- `float`，单精度浮点数
		- `double`，双精度浮点数
		- `repeated`，表示数组/列表
		- `message`，嵌套结构体
		- `enum`，枚举类型
	- **字段编号的作用：**
		- 如果没有字段编号，就像 JSON 一样，虽然对人类友好，但机器解析会更慢，因为需要扫描字符串来匹配字段名，而且数据中还要携带完整的字段名，导致占用更多空间。
		- Protobuf 仅传输字段编号和值，不传字段名，解析时根据编号映射到对应的字段名，既提升了解析效率，又减少了传输体积。
		- **不能随便更改字段编号：**
			- 如果随意更改字段编号，旧数据在反序列化时会将值映射到错误的字段，导致数据错乱，甚至程序崩溃。字段编号一旦确定，就必须保持不变，否则旧数据将无法正确解析。
			- **正确的做法是：**字段名可以修改，字段编号不能更改；若要删除字段，应保留原编号，避免重复使用。
- `protoc` 编译器
  heading:: true
	- 使用 `protoc` 编译器可以将 `.proto` 文件编译为对应语言（如 C++、Java、Python、Go 等）的代码，生成的代码可以直接导入并使用。
	- **安装 `protoc` 编译器：**
		- [下载预编译的压缩包](https://github.com/protocolbuffers/protobuf/releases)
		  logseq.order-list-type:: number
		- 找个位置解压
		  logseq.order-list-type:: number
		- 把 `bin` 目录添加到环境变量
		  logseq.order-list-type:: number
	- **编译：**
		- **Python：**
			- **生成 Python 代码：**`protoc --python_out=. person.proto`
			- **安装 Python 的 Protobuf 库：**`pip install protobuf`
-