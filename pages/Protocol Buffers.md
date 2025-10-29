概念
heading:: true
	- 简称 Protobuf，是 Google 开发的一种与语言无关的，用于定义数据结构和服务接口的工具。它根据 proto 文件中的定义，生成多种编程语言对应的数据结构操作代码（进行序列化与反序列化），支持在不同服务之间传输结构化数据，从而实现跨语言和系统的数据通信。
	- 相较于 JSON/XML，它是一种更高效的数据通信格式。
- `.proto` 文件
  heading:: true
	- **作用：**在 `.proto` 文件中使用特殊的语法（接口定义语言）来定义数据结构和服务接口。它就像是一张“合同”，告诉客户端和服务端要怎么通信、数据格式是怎样的。
	- **例子：**
		- ```protobuf
		  syntax = "proto3";
		  
		  package helloworld.v1;
		  
		  import "google/api/annotations.proto";
		  
		  option go_package = "user-management-system/api/helloworld/v1;v1";
		  option java_multiple_files = true;
		  option java_package = "dev.kratos.api.helloworld.v1";
		  option java_outer_classname = "HelloworldProtoV1";
		  
		  // The greeting service definition.
		  service Greeter {
		    // Sends a greeting
		    rpc SayHello (HelloRequest) returns (HelloReply) {
		      option (google.api.http) = {
		        get: "/helloworld/{name}"
		      };
		    }
		  }
		  
		  // The request message containing the user's name.
		  message HelloRequest {
		    string name = 1;
		  }
		  
		  // The response message containing the greetings
		  message HelloReply {
		    string message = 1;
		  }
		  ```
	- **`repeated` 字段：**
		- `repeated` 字段在 Go 中会生成一个对应类型的切片。
		- ```proto
		  message Article {
		    string title = 1;
		    repeated string tags = 2; // 文章可以有多个标签
		  }
		  ```
		- 生成的 Go `struct` 中会有一个 `Tags` 字段，会被导出（首字母大写），类型为 `[]string`。
	- **`optional` 关键字：**
		- 使用 `optional` 关键字后，生成的 Go `struct` 字段会是一个指针类型。这样，你就可以通过检查指针是否为 `nil` 来判断字段是否被显式设置。
	- **消息类型的嵌套：**
		- 在 Go 中，如果一个 `message` 字段引用了另一个 `message`，那么生成的 `struct` 字段会是一个指向被引用 `struct` 的指针。
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
	- **`go_package` 选项：用于指定生成的 Go 代码的包信息**
		- **这个选项分为两部分（用分号分隔）：**
			- **分号前（导入路径）：**定义其它 Go 程序 `import` 此包时使用的路径。这个路径必须是从你的 Go module 根目录开始的绝对路径。
			- **分号后（包名）：**写入到生成的 `.pb.go` 文件顶部的包声明 `package PACKAGE_NAME`。一个非常好的实践是在名称后加上 `pb` 后缀，以明确表示这是一个 Protobuf 生成的包。
		- **示例：**
			- ```proto
			  option go_package = "github.com/my-org/my-project/gen/go/user;userpb";
			  ```
			- 当 `protoc` 处理这个文件时，它知道：
				- 这个包的完整导入路径是 `github.com/my-org/my-project/gen/go/user`。
				- 生成的 `user.pb.go` 文件应该包含 `package userpb`。
		- **最佳实践：**
			- 将不同的服务放在不同的包中：
				- 不要这样：`option go_package = "user-service/api/rbac/v1;v1";`
				- 而是这样：`option go_package = "user-service/api/rbac/v1/permission;permissionpb";`
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
				- Google 将各语言的代码生成逻辑拆分为独立的语言插件。`protoc` 本身不直接生成 Go 代码，它需要通过插件来完成：
					- **`protoc-gen-go`：**用于解析 `.proto` 文件并生成对应的 `.pb.go` 源代码，包含 Go 的数据结构和序列化/反序列化方法。
					- **`protoc-gen-go-grpc`：**用于生成 gRPC 服务的客户端代码（用于发起 RPC 调用）、服务器端接口代码以及注册服务用的函数。
					- **`protoc-gen-go-http`：**
						- 将一个 HTTP 请求映射到 gRPC 方法，使得浏览器或前端也能通过 HTTP 请求访问你的服务。
						- `protoc-gen-go-http` 会为带有 HTTP 注解的 gRPC 方法生成中间代码，使服务在运行时能自动接收并处理 HTTP 请求：它先根据请求路径匹配路由，提取 URL、查询参数或请求体中的数据，构造 gRPC 请求结构体，调用对应的服务方法，最后将结果转为 JSON 返回给前端，从而实现 HTTP 到 gRPC 的自动衔接。
					- **适用场景：**
						- 如果只想用 Protobuf 传输结构化数据，不涉及 RPC，只需安装 `protoc-gen-go`。
						- 如果要用 gRPC 构建远程调用服务，则需要同时安装并使用 `protoc-gen-go` 和 `protoc-gen-go-grpc`，前者负责生成数据结构，后者负责生成服务接口。
						- 若需同时支持 HTTP 调用，只需加上 `protoc-gen-go-http`，即可让 gRPC 服务自动具备 REST 接口能力。
				- **安装命令：**
					- ```bash
					  go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
					  go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
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
			- **编译 `.proto` 文件：**
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
- **在 proto 文件中添加 HTTP 注解以启用 HTTP 支持：**
	- 主要目的是使同一个 `.proto` 定义能够同时生成 gRPC 服务端/客户端代码以及 HTTP/JSON 反向代理网关，从而让 gRPC 服务能够通过 HTTP RESTful API 访问。
	- 你可以在 `.proto` 文件中定义 gRPC 服务，并在服务的方法（RPC）上使用特殊的 `option` 添加 HTTP 注解，指明该 RPC 如何映射到 HTTP/JSON 接口。在你的 Go 应用中，同时启动 gRPC 服务和 HTTP 反向代理。当 HTTP 请求到来时，反向代理会将其转换为 gRPC 请求并发送给 gRPC 服务，随后将 gRPC 服务的响应转换为 HTTP/JSON 格式返回给客户端。
	- **在 `.proto` 文件中，添加 `google.api.http` 选项：**
		- ```proto
		  // 1. 导入 Google API 的注解定义
		  import "google/api/annotations.proto";
		  
		  // Greeter 服务定义
		  service GreeterService {
		    // SayHello 方法
		    rpc SayHello(SayHelloRequest) returns (SayHelloResponse) {
		      // 2. 添加 HTTP 注解
		      option (google.api.http) = {
		        // 定义一个 GET 请求，路径是 /v1/hello/{name}
		        // {name} 会自动映射到 SayHelloRequest 中的 name 字段
		        get: "/v1/hello/{name}"
		      };
		    }
		  
		    // CreateGreeting 方法
		    rpc CreateGreeting(CreateGreetingRequest) returns (SayHelloResponse) {
		      // 2. 添加另一个 HTTP 注解
		      option (google.api.http) = {
		        // 定义一个 POST 请求，路径是 /v1/greetings
		        // body: "*" 表示请求的 JSON body 会被映射到 CreateGreetingRequest 的所有字段
		        post: "/v1/greetings"
		        body: "*"
		      };
		    }
		  }
		  ```
- **为什么在 `.proto` 文件中定义的是 `int64` 类型，返回的却是字符串？**
	- 这不是 Bug。将 `int64` 序列化为 JSON 字符串是 Protobuf 的标准做法，目的是避免在 JavaScript 中出现精度丢失。
	- **问题原因：**
		- 为了确保数据在不同系统间的兼容性和完整性，Kratos 的 HTTP Server（即 gRPC-Gateway 实现）会对某些类型的数据进行转换。
		- **JavaScript 的数字精度限制：**JavaScript 中的 `Number` 类型基于 IEEE 754 双精度浮点数，只能安全表示范围在 `-(2^53 - 1)` 到 `(2^53 - 1)` 之间的整数，即 `Number.MAX_SAFE_INTEGER = 9007199254740991`。
		- **Protobuf 的 `int64` 类型：**能表示的整数范围远大于 JavaScript，达到 `-9223372036854775808` 到 `9223372036854775807`。
		- **数据丢失风险**：如果 gRPC-Gateway（Kratos 中的 HTTP Server 就是一个实现）直接将超出 JavaScript 安全范围的 `int64` 值（例如 `9007199254740992`）序列化为 JSON 数字，那么在浏览器或其他 JavaScript 环境中解析该 JSON 时，就会因精度丢失而得到不准确的数值。
		- **官方规范：**为避免上述问题，Protobuf 的 JSON 映射规范（proto3-spec）明确规定：`int64` 和 `uint64` 类型在序列化为 JSON 时，必须以十进制字符串形式表示。
		- 因此，你看到的 `"code": "0"`、`"total": "2"` 等字段，是 Kratos 框架在 HTTP Server/Gateway 层根据规范自动完成的转换，这并非发生在你编写的 `service`、`biz` 或 `data` 层代码中。
	- **解决方案：**
		- **方案一：修改 `.proto` 定义**
			- 为字段选择合适的数据类型：这些字段是否真的需要用到 `int64` 这么大的范围？
			- 如果不需要，可以将 `.proto` 文件中的字段类型改为 `int32`，这样在序列化为 JSON 时会被标准地表示为数字类型（number）。
		- **方案二：由客户端进行适配**
			- 如果字段确实可能是非常大的数，必须使用 `int64`，那么将其序列化为字符串是符合规范的行为。
			- 此时，应由 API 的消费者（例如前端的 JavaScript 代码）负责处理这种字符串格式的数字。前端拿到 `"total": "2"` 这类数据时，应使用 `parseInt()` 或 `Number()` 将其转换为数字再进行后续处理。
-
-