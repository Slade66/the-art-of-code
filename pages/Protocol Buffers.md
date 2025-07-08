概念
heading:: true
	- 简称 Protobuf，是 Google 开发的一种与语言无关、平台无关、具有良好可扩展性的序列化数据结构机制。
	- Protobuf 是用于定义数据结构并对数据进行序列化和反序列化的工具，相较于 JSON/XML，它是一种更高效的数据通信格式，用于在不同服务之间传输结构化数据（通信）。
	- Protobuf 是一种工具，用于定义数据结构，并根据这些定义生成多种编程语言对应的数据结构操作代码，从而实现跨语言的数据通信。
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
		- 如果不用字段编号，像 JSON 一样，虽然对人类友好，但机器解析会更慢，因为需要扫描字符串来匹配字段名，而且数据中还要携带完整的字段名，导致占用更多空间。
		- Protobuf 仅传输字段编号和值，不传字段名，解析时根据编号映射到对应的字段名，既提升了解析效率，又减少了传输体积。
		- **不能随便更改字段编号：**
			- 如果随意更改字段编号，旧数据在反序列化时会将值映射到错误的字段，导致数据错乱，甚至程序崩溃。字段编号一旦确定，就必须保持不变，否则旧数据将无法正确解析。
			- **正确的做法是：**字段名可以修改，字段编号不能更改；若要删除字段，应保留原编号，避免重复使用。
- `protoc` 编译器
  heading:: true
	- 使用 `protoc` 编译器可以将 `.proto` 文件编译为对应语言（如 C++、Java、Python、Go 等）的代码，生成的代码可以直接导入并使用。
	- **安装 `protoc` 编译器：**
	  id:: 686c8e4e-cbc7-4781-9575-a77cbc8d2372
		- [下载预编译的压缩包](https://github.com/protocolbuffers/protobuf/releases)
		  logseq.order-list-type:: number
		- 找个位置解压
		  logseq.order-list-type:: number
		- 把 `bin` 目录添加到环境变量
		  logseq.order-list-type:: number
	- **编译 & 使用：**
		- **Python：**
			- **生成 Python 代码：**`protoc --python_out=. person.proto`
				- 生成的文件以 `_pb2.py` 结尾，这是 Google 官方的命名约定，用于表示通过 Protobuf 编译器自动生成的 Python 模块，可直接导入使用。
				- `pb` 代表 Protocol Buffer
				- `2` 表示使用的是版本 2 的 API（支持 proto2 和 proto3）
			- **安装 Python 的 Protobuf 库：**`pip install protobuf`
			- **在 Python 中使用 Protobuf 消息：**
				- ```python
				  import person_pb2
				  
				  # 创建一个 Person 对象
				  p = person_pb2.Person()
				  p.name = "小泽"
				  p.id = 123
				  p.email = "xiaoze@school.edu"
				  p.phones.extend(["13800000000", "13900000000"])  # 添加多个手机号
				  
				  # 打印对象内容（文本格式）
				  print("原始对象：")
				  print(p)
				  
				  # 序列化成二进制
				  binary_data = p.SerializeToString()
				  
				  # 反序列化回对象
				  p2 = person_pb2.Person()
				  p2.ParseFromString(binary_data)
				  
				  print("\n反序列化后的对象：")
				  print(p2)
				  ```
		- **Go：**
			- **安装 `protoc-gen-go`：**
				- Google 将各语言的代码生成逻辑拆分为独立的语言插件。
				- `protoc-gen-go` 负责解析 `.proto` 文件并生成对应的 `.pb.go` 源代码。
				- **`protoc-gen-go` 与 `protoc-gen-go-grpc` 的区别：**
					- **protoc-gen-go：**根据 `.proto` 文件生成 Go 的数据结构代码（消息类型），即 `.pb.go` 文件，包含对应的结构体以及序列化和反序列化方法。
					- **protoc-gen-go-grpc：**生成 gRPC 服务的接口定义、服务端实现骨架和客户端调用代码，方便实现和调用 gRPC 服务。
					- **适用场景：**
						- 如果只想用 Protobuf 传输结构化数据，不涉及 RPC，只需安装 `protoc-gen-go`。
						- 如果要用 gRPC 构建远程调用服务，则需要同时安装并使用 `protoc-gen-go` 和 `protoc-gen-go-grpc`，前者负责生成数据结构，后者负责生成服务接口。
				- **安装命令：**
					- ```bash
					  go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
					  ```
			- **编写 `.proto` 文件：**
				- ```proto
				  syntax = "proto3";
				  
				  option go_package = "./pb;pb";  // 指定生成的 Go 包路径（必填）
				  
				  message Person {
				    string name = 1;
				    int32 id = 2;
				    string email = 3;
				  }
				  ```
			- **生成 `.pb.go` 文件：**
				- **运行命令：**
					- ```bash
					  protoc --go_out=. person.proto
					  ```
				- 该命令会生成一个 `person.pb.go` 文件，包含 `Person` 结构体定义及其序列化、反序列化方法。
			- **添加 Protobuf 运行时库：**
				- ```bash
				  go get google.golang.org/protobuf@latest
				  ```
				- 该库包含了 Protobuf 的核心 Go 实现，包括消息的接口定义、序列化和反序列化方法等。
			- **在 Go 代码中使用：**
				- ```go
				  package main
				  
				  import (
				  	"fmt"
				  	"log"
				  
				  	"google.golang.org/protobuf/proto"
				  
				  	"根包名/pb"
				  )
				  
				  func main() {
				  	// 创建一个 Person 对象
				  	p := &pb.Person{
				  		Name:  "小泽",
				  		Id:    1001,
				  		Email: "xiaoze@school.edu",
				  	}
				  
				  	// 序列化
				  	data, err := proto.Marshal(p)
				  	if err != nil {
				  		log.Fatalf("序列化失败: %v", err)
				  	}
				  
				  	// 反序列化
				  	newP := &pb.Person{}
				  	err = proto.Unmarshal(data, newP)
				  	if err != nil {
				  		log.Fatalf("反序列化失败: %v", err)
				  	}
				  
				  	fmt.Println("解码后：", newP)
				  }
				  ```
-