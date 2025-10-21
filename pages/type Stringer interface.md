- **源码：**
	- ```go
	  // Stringer is implemented by any value that has a String method,
	  // which defines the “native” format for that value.
	  // The String method is used to print values passed as an operand
	  // to any format that accepts a string or to an unformatted printer
	  // such as [Print].
	  type Stringer interface {
	  	String() string
	  }
	  ```
- **作用：**
	- 让你自定义类型的打印方式。
	- 默认的打印格式往往难以准确表达其含义。当你为类型实现 `String() string` 方法后，就相当于告诉 `fmt` 包：“当需要把我的值转换为字符串时，请调用我的 `String()` 方法，由我来决定打印的样子。”
- **示例：**
	- ```go
	  package main
	  
	  import "fmt"
	  
	  // Device 表示一个网络设备
	  type Device struct {
	  	Hostname string
	  	IP       string
	  	Vendor   string
	  }
	  
	  // String 方法让 Device 类型实现了 fmt.Stringer 接口
	  func (d Device) String() string {
	  	// 我们自定义一个更具可读性的格式
	  	// 注意：这里我们使用 Sprintf 来格式化字符串并返回
	  	// Sprintf 不会触发打印，所以是安全的
	  	return fmt.Sprintf("[%s] %s (%s)", d.Vendor, d.Hostname, d.IP)
	  }
	  
	  func main() {
	  	d := Device{
	  		Hostname: "core-router-01",
	  		IP:       "192.168.1.1",
	  		Vendor:   "Huawei",
	  	}
	  
	  	// 使用 fmt.Println 打印
	  	fmt.Println("设备信息:", d)
	  }
	  
	  ```
	- ```
	  设备信息: {core-router-01 192.168.1.1 Huawei} // 没实现 fmt.Stringer 接口前
	  设备信息: [Huawei] core-router-01 (192.168.1.1) // 实现了 fmt.Stringer 接口后
	  ```
-