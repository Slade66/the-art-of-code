## `uintptr` 是什么？
	- `uintptr` 是一个足够大、能存下指针地址的无符号整数。
- ## 为什么需要 `uintptr`？
	- 用于指针算术运算（Pointer Arithmetic）。
	- Go 的设计原则是内存安全，因此 Go 语言中普通的指针（如 `*int`）和通用指针（`unsafe.Pointer`）都不支持算术运算（比如地址加减），防止你越界访问内存。
	- 但在某些底层操作中（例如与操作系统交互、Cgo、或者手动操作内存布局），我们需要计算内存地址。这时就需要将指针转换为 `uintptr` 进行整数运算。
- ## Go 的指针体系
	- | **类型** | **描述** | **特点** | **是否被 GC 追踪** | **支持运算** |
	  | ---- | ---- | ---- |
	  | **`*T`** | 类型安全指针 | 只能指向特定类型（如 `*int`）。 | **是** | 否 |
	  | **`unsafe.Pointer`** | 通用指针 | 类似于 C 的 `void*`。可以在不同类型的指针间转换。 | **是** | 否 |
	  | **`uintptr`** | **地址整数** | **只是一个数字**。保存了内存地址的值。 | **否** | **是** |
- ## 代码示例
	- **使用流程：**
		- 将 `*T` 转为 `unsafe.Pointer`。
		- 将 `unsafe.Pointer` 转为 `uintptr`。
		- 对 `uintptr` 进行加减运算（指针运算）。
		- 将运算后的 `uintptr` 转回 `unsafe.Pointer`。
		- 最后转回 `*T` 访问新地址的值。
	- **通过指针运算访问结构体字段：**
		- 绕过字段名，直接通过内存偏移量修改结构体字段。
		- ```go
		  package main
		  
		  import (
		  	"fmt"
		  	"unsafe"
		  )
		  
		  type User struct {
		  	Name string
		  	Age  int
		  }
		  
		  func main() {
		  	u := User{Name: "Alice", Age: 18}
		  
		  	// 1. 获取结构体的起始指针
		  	ptr := unsafe.Pointer(&u)
		  
		  	// 2. 计算 Age 字段的内存地址
		  	// Age 的地址 = 结构体起始地址 + Age 的偏移量
		  	// 注意：这里必须将 Pointer 转为 uintptr 才能做加法
		  	ageOffset := unsafe.Offsetof(u.Age)
		  	agePtrVal := uintptr(ptr) + ageOffset
		  
		  	// 3. 将计算出的整数地址转回指针
		  	agePtr := (*int)(unsafe.Pointer(agePtrVal))
		  
		  	// 4. 修改值
		  	*agePtr = 25
		  
		  	fmt.Println(u) // 输出: {Alice 25}
		  }
		  ```
		- **`User` 结构体的内存布局：**
			- | **相对地址 (Offset)** | **字段** | **内容** | **大小** | **说明** |
			  | ---- | ---- | ---- |
			  | **0 - 7** | `Name.Data` | `0x...` (指针) | 8 byte | 指向 "Alice" 的内存地址 |
			  | **8 - 15** | `Name.Len` | `5` | 8 byte | 字符串长度 |
			  | **16 - 23** | **`Age`** | `18` | 8 byte | int 值 |
		- **为什么 `ageOffset` 是 16？**
			- 因为你的程序运行在 64 位架构（amd64） 的操作系统上。
			- 在 64 位系统中，Go 语言的 `string` 类型占用 16 个字节。由于 `Age` 紧挨着 `Name`，所以 `Age` 的偏移量（Offset）就是 `Name` 占用的这 16 个字节。
		- **为什么 `string` 占 16 字节？**
			- 在 Go 的底层（runtime），`string` 是一个包含两个字段的小结构体（Struct），称为 `StringHeader`。
			- ```go
			  // 伪代码表示 Go 内部的 string 结构
			  type StringHeader struct {
			      Data uintptr // 指向底层字节数组的指针 (8 字节)
			      Len  int     // 字符串的长度 (8 字节)
			  }
			  ```
			- 在 64 位系统中：
				- 指针 (`Data`)：地址长度为 64 位，占用 8 字节。
				- 整数 (`Len`)：`int` 在 64 位系统中也是 8 字节。
				- 合计：8 + 8 = 16 字节。
			- **注意：**真正的字符串内容（"Alice"）是存储在堆上的其他位置的，不占用这个结构体的空间。
- ## 注意
	- 除非你要写极低层的库（如 runtime、os 交互），否则不要在业务代码中使用它。
	- `uintptr` 只是一个整数，Go 的垃圾回收器（GC）不会把它当成指针。
		- 如果一个对象仅被一个 `uintptr` 变量引用（没有其他的 `*T` 或 `unsafe.Pointer` 指向它），GC 可能会回收该对象。
-