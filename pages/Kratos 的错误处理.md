#### 核心原则：错误要进协议文件（`.proto`），不要到处 `errors.New("xxx")`。
	- 错误也是 API 的一部分，API 不仅要定义成功时的数据格式，还要明确失败时返回什么。
	- 将错误定义集中在一份协议文件（如 `.proto`）中作为唯一真相来源（Single Source of Truth），通过代码生成器为各语言生成对应的错误常量和构造函数。无论后端有多少服务、使用 Go 还是 Java，都能共享相同的错误常量和错误码，避免拼写错误。
	- 统一定义、统一生成，让错误在跨服务、跨语言间保持一致性和可识别性，减少维护成本，避免因错误码不统一导致的调试困难，并确保客户端和服务端对错误的理解始终一致。
- #### 在 .proto 中如何定义错误？
	- 在 `.proto` 里新建一个 `enum`，列出所有业务错误的名称（Reason）。
	- ```proto
	  syntax = "proto3";
	  
	  import "errors/errors.proto";
	  
	  enum ErrorReason {
	    option (errors.default_code) = 500; // 默认 HTTP 状态码
	    IMAGE_BUILD_FAILED        = 0 [(errors.code) = 500]; // 镜像构建失败
	    DOCKER_CONNECTION_FAILED  = 1 [(errors.code) = 502]; // 连接 Docker Daemon 失败
	    HOST_NOT_FOUND            = 2 [(errors.code) = 404]; // 主机未找到
	    INVALID_BUILD_ARGUMENT    = 3 [(errors.code) = 400]; // 无效的构建参数
	  }
	  ```
- #### protoc 生成的产物
	- 搭配 `protoc-gen-go-errors` 插件，`protoc` 会为每个枚举成员生成：
		- 常量（Reason 名称）
		- 带 HTTP 状态码的错误构造函数
		- 判断函数（`IsXxx(err)`）
	- **生成后的 Go 代码：**
		- ```go
		  const ReasonImageBuildFailed = "IMAGE_BUILD_FAILED"
		  
		  func ErrorImageBuildFailed(format string, args ...any) *errors.Error {
		      return errors.New(500, ReasonImageBuildFailed, fmt.Sprintf(format, args...))
		  }
		  
		  func IsImageBuildFailed(err error) bool {
		      return errors.Reason(err) == ReasonImageBuildFailed
		  }
		  ```
	- **业务里用法：**
		- ```go
		  // 返回构建失败
		  return nil, v1.ErrorImageBuildFailed("build failed on host %s", hostID)
		  
		  // 判断错误类型
		  if v1.IsHostNotFound(err) {
		      // 特殊处理
		  }
		  ```
-