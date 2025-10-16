概念
heading:: true
	- gRPC（Google Remote Procedure Call）是 Google 开发的一个高性能、开源的远程过程调用（RPC）框架。
	- gRPC 在你调用远程函数时，把：函数调用 => 网络请求 => 序列化/反序列化 => 网络通信 => 响应返回，这一整套复杂过程，打包得像调用本地方法一样简单。
	- gRPC 是一个用 `.proto` 定义接口和数据格式，然后自动生成服务端接口 + 客户端调用代码 + 底层通信机制的远程调用框架。
	- **gRPC 生成的代码：**
		- gRPC 会根据 `.proto` 文件中定义的接口和数据结构，自动生成客户端和服务端的通信代码。服务端只需实现方法逻辑（如查询数据库），客户端则像调用本地函数一样发起远程调用。
		- **服务端接口代码：**gRPC 会生成一个服务端接口，你只需实现其中定义的方法。
		- **客户端代码：**gRPC 帮你生成了客户端代理对象，你可以直接通过这个对象调用远程函数。
		- **此外，还会生成：**
			- Protobuf 消息类型（即请求/响应的结构体）
			- gRPC 注册、序列化、网络底层代码
	- **类比：**
		- 你写的 `.proto` 文件相当于：
			- ```java
			  interface UserService {
			      UserResponse getUser(UserRequest req);
			  }
			  ```
		- **然后：**
			- 服务端去实现这个接口
			- 客户端拿到一个代理类去调用这个方法（底层自动帮你发请求）
		- gRPC 就是自动生成接口、消息类型、客户端代理，并处理好底层的网络通信。
	- **高性能：**gRPC 的网络传输基于 HTTP/2 协议，支持多路复用、长连接、头部压缩和流式传输。
	- **跨语言：**
		- 支持多种编程语言，包括 Go、Java、Python、C++、Node.js、Rust 等。
		- 服务端可以使用 Go 编写，客户端则可以使用 Java、Node 或 Python 调用。各语言之间通过同一份 `.proto` 文件来保证数据结构和接口的一致性，你只需维护这一份 `.proto` 文件，其它语言即可通过对应的插件自动生成所需的代码。
	- **远程调用：**使微服务之间的通信像调用本地函数一样简单，客户端可以像调用本地方法一样直接调用远程服务器上的方法，底层的网络通信由框架自动处理。
	- **使用 Protocol Buffers：**gRPC 使用 Protobuf 来定义消息数据结构（格式）和服务接口，然后自动生成代码。
- 使用
  heading:: true
	- 安装环境
	  heading:: true
		- ((686c8e4e-cbc7-4781-9575-a77cbc8d2372))
		- **安装 Go 插件：**
			- 用于生成 Go 语言专用的 protobuf 和 gRPC 代码。
			- ```bash
			  go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
			  go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
			  ```
		- **安装 `google.golang.org/grpc` 包：**
			- 用于构建 gRPC 服务器和客户端程序。
			- ```bash
			  go get google.golang.org/grpc
			  ```
	- 定义服务
	  heading:: true
		- ```proto
		  syntax = "proto3"; // 使用 proto3 语法版本
		  
		  package user; // 定义 proto 包名
		  
		  // 指定生成的 Go 包路径，生成的代码会放在 userpb 文件夹中
		  option go_package = "/userpb"; 
		  
		  // 定义 gRPC 服务
		  service UserService {
		    // 定义一个远程调用方法：GetUser，接收 UserRequest，返回 UserResponse
		    rpc GetUser (UserRequest) returns (UserResponse);
		  }
		  
		  // 请求消息体，携带一个用户 ID
		  message UserRequest {
		    int32 id = 1;
		  }
		  
		  // 响应消息体，包含用户基本信息
		  message UserResponse {
		    int32 id = 1;
		    string name = 2;
		    string email = 3;
		  }
		  ```
	- 编译生成 Go 代码
	  heading:: true
		- ```bash
		  protoc --go_out=. --go-grpc_out=. user.proto
		  ```
		- `--go_out=.`：生成消息结构体（UserRequest、UserResponse）
		- `--go-grpc_out=.`：生成服务接口（UserServiceServer 接口、注册函数）
	- 实现服务端
	  heading:: true
		- ```go
		  package main
		  
		  import (
		      "context"
		      "log"
		      "net"
		  
		      "test-go/userpb" // 引入编译生成的 proto 包
		      "google.golang.org/grpc"
		  )
		  
		  // 定义服务端结构体，继承生成的接口（必须嵌入 UnimplementedXXX 才能编译）
		  type server struct {
		      userpb.UnimplementedUserServiceServer
		  }
		  
		  // 实现服务端的 GetUser 方法
		  func (s *server) GetUser(ctx context.Context, req *userpb.UserRequest) (*userpb.UserResponse, error) {
		      log.Printf("Received request for user ID: %d", req.Id)
		  
		      // 返回一个 UserResponse（模拟数据库数据）
		      return &userpb.UserResponse{
		          Id:    req.Id,
		          Name:  "小泽",
		          Email: "xiaoze@example.com",
		      }, nil
		  }
		  
		  func main() {
		      // 监听本地 50051 端口（gRPC 默认端口）
		      lis, err := net.Listen("tcp", ":50051")
		      if err != nil {
		          log.Fatalf("failed to listen: %v", err)
		      }
		  
		      // 创建 gRPC 服务器实例
		      grpcServer := grpc.NewServer()
		  
		      // 注册服务到 gRPC 服务器
		      userpb.RegisterUserServiceServer(grpcServer, &server{})
		  
		      log.Println("gRPC server running on :50051")
		  
		      // 启动服务，开始监听客户端请求
		      if err := grpcServer.Serve(lis); err != nil {
		          log.Fatalf("failed to serve: %v", err)
		      }
		  }
		  ```
	- 实现客户端
	  heading:: true
		- ```go
		  package main
		  
		  import (
		      "context"
		      "log"
		      "time"
		  
		      "test-go/userpb" // 引入编译生成的 proto 包
		      "google.golang.org/grpc"
		  )
		  
		  func main() {
		      // 建立连接到 gRPC 服务端
		      conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure()) // ⚠️ 生产环境应使用 TLS
		      if err != nil {
		          log.Fatalf("could not connect: %v", err)
		      }
		      defer conn.Close()
		  
		      // 创建客户端 stub（代理）
		      client := userpb.NewUserServiceClient(conn)
		  
		      // 创建上下文，设置超时时间
		      ctx, cancel := context.WithTimeout(context.Background(), time.Second)
		      defer cancel()
		  
		      // 远程调用 GetUser 方法
		      resp, err := client.GetUser(ctx, &userpb.UserRequest{Id: 1})
		      if err != nil {
		          log.Fatalf("error calling GetUser: %v", err)
		      }
		  
		      // 打印返回的用户信息
		      log.Printf("User: %v", resp)
		  }
		  ```
- ## gRPC 的四种模式
	- **Unary RPC（普通请求-响应）**
		- 类似 HTTP，一问一答：客户端发送一次请求，服务端返回一次结果。
	- **Server-side Streaming RPC（服务端流）**
		- 客户端发一次请求，服务端持续推送多条消息，直到任务结束。
		- **适用场景：**
			- 构建等长耗时任务无法立即返回结果，需要实时反馈，让前端能够随时展示进度。
			- 一次请求获取全部日志，避免前端频繁轮询。
	- **Client-side Streaming RPC（客户端流）**
		- 客户端分多次发送消息，服务端在接收完毕后统一返回结果。
	- **Bidirectional Streaming RPC（双向流）**
		- 客户端与服务端可同时、持续地发送和接收消息，边发边收，互不阻塞。
	- **proto 示例：**
		- ```proto
		  service BuildService {
		    // 一元 RPC
		    rpc Ping(PingReq) returns (PingResp);
		    // 服务端流
		    rpc Build(BuildReq) returns (stream BuildLog);
		    // 客户端流
		    rpc Upload(stream Chunk) returns (UploadResult);
		    // 双向流
		    rpc Chat(stream ChatMsg) returns (stream ChatMsg);
		  }
		  ```
-