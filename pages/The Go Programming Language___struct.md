- **结构体是什么？**
  collapsed:: true
	- 数组、切片和映射用于存储一组相同类型的数据，而结构体则用于将多个不同类型的值组合成一个整体。
	- 结构体是由若干不同类型字段组成的数据集合。允许你把多个不同类型的数据“打包”在一起，形成一条记录。
- **结构体的类型定义与变量声明**
  collapsed:: true
	- **匿名结构体（一次性使用）**
		- **是什么？**匿名结构体就是一种定义的时候没有名称的结构体类型。
		- **有什么用？**
			- 如果一个结构体类型只用于一个值，不想给它取名，不值得专门为它定义一个类型时。
			- 它适合作为临时的数据容器，用来快速存放一组相关字段，而无需额外声明具名结构体。
			- 常用于表格驱动的测试。
		- **定义和使用匿名结构体**
			- ```go
			  package main
			  
			  import "fmt"
			  
			  func main() {
			  	lyz := struct {
			  		Name string
			  		Age  int
			  	}{
			  		Name: "lyz",
			  		Age:  24,
			  	}
			  	fmt.Println(lyz)
			  }
			  ```
			- ```go
			  package main
			  
			  import "fmt"
			  
			  func main() {
			  	var lyz struct {
			  		Name string
			  		Age  int
			  	}
			  	lyz.Name = "lyz"
			  	lyz.Age = 24
			  	fmt.Println(lyz)
			  }
			  
			  ```
	- **具名结构体类型定义及变量声明（多次使用）**
		- 结构体类型通过 `type` 关键字来定义。
		- **语法：**
			- ```go
			  type 类型名 struct {
			    字段名 类型
			    字段名 类型
			  }
			  ```
		- **示例：**
			- ```go
			  type person struct {
			    age   int
			    money float64
			  }
			  
			  var lyz person
			  ```
	- 声明结构体变量后，其所有字段都会被自动初始化为对应类型的零值。
- **访问结构体字段**
  collapsed:: true
	- 使用 `.` 运算符访问（读取、写入）结构体字段。
		- ```go
		  lyz.age = 23
		  fmt.Println(lyz.money)
		  ```
	- 你可以直接使用结构体变量调用指针接收者的方法，Go 会自动帮你取地址：
		- ```go
		  type Person struct {
		      age int
		  }
		  
		  // 指针接收者方法
		  func (p *Person) GrowUp() {
		      p.age += 1
		  }
		  
		  func main() {
		      lyz := Person{age: 23}
		  
		      // 调用指针接收者方法，Go 自动取地址
		      lyz.GrowUp()
		  
		      fmt.Println(lyz.age) // 输出 24
		  }
		  ```
- **结构体的指针**
  collapsed:: true
	- 结构体指针是一个持有结构体变量的内存地址的指针变量。
	- **直接把结构体传给函数：**
		- `struct` 是值类型，传递给函数时会进行拷贝，因此在函数中修改不会影响原始结构体，同时也存在性能开销。
		- 为了修改原始数据，也为了提高性能，通常传递结构体的指针给函数或方法。
	- **把结构体的指针传给函数是为了：**节省内存、提高性能并允许修改原数据。
	- **Go 允许在 `nil` 指针上调用指针接收者的方法：**
		- 在方法内部，指针接收者的变量为 `nil`，只要内部逻辑对这种情况进行适当处理，就可以确保安全。
	- **自动解引用：**
		- **理论上的"标准"做法：**
			- 我们有一个指针 `p`。
			- 为了获取 `p` 指向的那个*值*（也就是 `v` 结构体本身），我们必须先**解引用** 指针，写作 `(*p)`。
			- `(*p)` 现在代表的就是 `v` 结构体。
			- 为了访问它的 `X` 字段，我们使用点号 `.`，所以完整的写法是 `(*p).X`。
			- **注意：** 这里的括号 `()` 是**必须的**。如果不加，`*p.X` 会被 Go 编译器理解为 `*(p.X)`，它会尝试访问一个叫 `X` 的字段... 但 `p` 是个指针，不是结构体，它没有字段，所以这会编译错误。
		- **Go 语言的"便捷"做法 (语法糖)：**
			- Go 的设计者认为 `(*p).X` 这种显式解引用的写法过于繁琐，因此在结构体指针的字段访问上提供了语法糖。
			- 不同于 C++ 必须写成 `(*ptr).field` 或 `ptr->field`，在 Go 中你可以直接使用 `ptr.field`。
			- 当你对结构体指针使用 `.` 访问字段时（例如 `p.X`），编译器会自动将其转换为 `(*p).X`。
			- 这个特性让你无需显式解引用，代码更简洁易读。
	- **惯用的写法：**
		- ```go
		  // 在内存中创建 Vertex 实例，紧接着获取这个刚刚被创建的实例的内存地址，把这个地址赋值给变量 p。
		  p = &Vertex{1, 2}
		  
		  // 这避免了你写两行代码：
		  temp_v := Vertex{1, 2} // 1. 创建实例
		  p := &temp_v           // 2. 获取指针
		  ```
- **使用结构体字面量创建结构体实例**
  collapsed:: true
	- 结构体字面量语法，让你能“字面上”就看出一个结构体实例长什么样，快速创建结构体变量并初始化赋值。
	- 若省略某些字段，则这些字段会自动初始化为对应类型的零值。
	- ```go
	  type Vertex struct { X, Y int } // 只是一个蓝图，它定义了 Vertex 这种类型应该长什么样。
	  
	  Vertex{1, 2} // 告诉 Go：“请在内存中帮我新分配一块地方，按照 Vertex 蓝图，把 X 字段填上 1，Y 字段填上 2。”
	  ```
	- Go 提供了两种方式来填写 `Type{...}` 里的值：
		- **方式一：按位置**
			- 你必须**严格按照** `type` 定义中字段的**顺序**来提供值。
			- **缺点：**
				- 你必须提供**所有**字段的值，一个都不能少。
				- 如果未来你修改了结构体的定义（比如多加了一个字段），结构体字面量创建对象就会失败。
			- 通常不推荐这种写法，因为它可读性差且难以维护。
		- **方式二：按名称**
			- `v2 = Vertex{X: 1}`
				- 你明确地告诉 Go：“我只想设置 `X` 字段的值为 `1`。”
				- 你没有提到 `Y` 字段，Go 会自动把 `Y` 设置为其**零值**。
			- **优势：**
				- **可读性强：**清楚地表明哪个值赋给哪个参数。
				- **顺序无关：**`Vertex{X: 1, Y: 2}` 和 `Vertex{Y: 2, X: 1}` 是**完全等价的**。
				- **健壮性高：**如果 `Vertex` 结构体未来增加了新字段 `Z`，`Vertex{X: 1}` 这行代码**仍然可以正常编译**，`Z` 会被自动设为零值。
			- **请始终使用 `Name:` 这种具名字段的语法**来初始化结构体。
- **嵌套类型省略**
  collapsed:: true
	- 如果内部元素的类型可以从外部推断出来，你就不需要在内部重复写类型名。
	- ```go
	  type Point struct {
	      X, Y int
	  }
	  
	  // 啰嗦的写法：
	  points := []Point{
	      Point{X: 1, Y: 2},
	      Point{X: 3, Y: 4},
	  }
	  
	  // 啰嗦的写法：
	  // Go 知道 []Point 里装的一定是 Point，所以里面的 Point 可以省略。
	  points := []Point{
	      {1, 2}, // 直接写 {} 即可
	      {3, 4},
	  }
	  
	  // 也同样适用于 Map 和多维切片：
	  // 啰嗦：map[string]Point{"A": Point{1, 2}}
	  // 简洁：
	  grid := map[string]Point{
	      "Home": {0, 0}, 
	      "Work": {10, 20},
	  }
	  ```
- **结构体的导出规则**
  collapsed:: true
	- 如果一个结构体类型在包外可见（即类型名首字母大写），但其字段名未大写（未导出），则这些字段在包外仍不可访问。字段要在包外可访问，需使用大写字母开头。
- **什么时候嵌入结构体变量，什么时候嵌入结构体指针？**
  collapsed:: true
	- 如果一个结构体为空，唯一的作用是提供方法，不存储任何数据或状态。对于没有状态的“函数集”，嵌入空结构体几乎没有开销。对于这种没有状态、仅提供方法的结构体，可以直接使用值嵌入的简单方式。
	- 对于有状态的实例，必须通过指针引用，而非复制。因为嵌入值会带来复制开销，并且如果原状态发生变化，复制对象无法同步更新。
- **结构体嵌入（Struct Embedding）**
  collapsed:: true
	- **什么是结构体嵌入？**
	  collapsed:: true
		- 它允许你将一个结构体直接作为另一个结构体的字段，而不需要给这个字段起名字。
		- 当你在一个结构体中声明一个字段，但只写了类型而没有写字段名时，这就是嵌入（也称为匿名嵌入）。
		- **隐式字段命名：**嵌入字段的名字 = 类型名。
			- 当你写 `type A struct { B }` 时，编译器实际上在背后把它看作：
				- ```go
				  type A struct {
				      B B  // 字段名是 B，类型也是 B
				  }
				  ```
				- 所谓的“直接塞进去”，只是省略了写名字这个步骤，编译器帮你补上了。
			- **硬性限制：**由于字段名是根据类型名自动生成的，你不能在同一个结构体中嵌入两个同名的类型（即使它们来自不同的包）。
			- **嵌入指针类型：**`*T` -> 字段名依然是 `T`（而不是 `*T`，字段名不包含符号）。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  // 定义一个基础结构体
		  type User struct {
		  	Name  string
		  	Email string
		  }
		  
		  // 定义一个方法
		  func (u *User) Notify() {
		  	fmt.Printf("Sending email to %s <%s>\n", u.Name, u.Email)
		  }
		  
		  // Admin 嵌入了 User
		  type Admin struct {
		  	User  // <--- 这就是嵌入，没有字段名
		  	Level string
		  }
		  
		  func main() {
		  	// 初始化
		  	admin := Admin{
		  		User: User{
		  			Name:  "Alice",
		  			Email: "alice@example.com",
		  		},
		  		Level: "Super",
		  	}
		  
		  	// 1. 直接访问 User 的字段（字段提升）
		  	fmt.Println("Admin Name:", admin.Name) // 等同于 admin.User.Name
		  
		  	// 2. 直接调用 User 的方法（方法提升）
		  	admin.Notify() // 等同于 admin.User.Notify()
		  }
		  
		  ```
	- **提升**
	  collapsed:: true
		- 内部字段和方法被“提升”到外部，可以直接访问。
		- **字段提升：**你可以直接通过外层结构体访问内层结构体的字段，仿佛它们是外层自己的一样。
		- **方法提升：**被嵌入结构体的方法，可以直接通过外部结构体调用。
	- **覆盖**
	  collapsed:: true
		- 如果外部结构体和内部结构体有同名的字段或方法，外部的会覆盖内部的。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  type Base struct {
		  	Status string
		  }
		  
		  type Container struct {
		  	Base
		  	Status string // 外部也有 Status
		  }
		  
		  func main() {
		  	c := Container{}
		  	c.Status = "Outer"      // 访问的是 Container.Status
		  	fmt.Println(c)          // {{} Outer}
		  	c.Base.Status = "Inner" // 必须显式指定才能访问内部的 Status
		  	fmt.Println(c)          // {{Inner} Outer}
		  }
		  
		  ```
		- **重写：**
			- 在 Go 里，其实并没有传统面向对象语言（比如 Java、C++）里那种 “重写（Override）” 的概念，但效果上可以做到。
			- Go 不是靠继承重写，而是靠组合 + 方法提升。
			- 本质不是替换嵌入结构体的方法，而是外部结构体有了一个同名方法，调用时编译器会优先选择“层级最浅”的那一个，也就是优先用外部结构体自己的那个。
			- ```go
			  package main
			  
			  import "fmt"
			  
			  // 1. 定义内部结构体
			  type Robot struct{}
			  
			  func (r *Robot) Work() {
			  	fmt.Println("Robot: I am working...")
			  }
			  
			  // 2. 定义外部结构体
			  type AdvancedRobot struct {
			  	Robot // 嵌入
			  }
			  
			  // 3. "重写" (遮蔽) Work 方法
			  func (a *AdvancedRobot) Work() {
			      // 如果你还想在“重写”里用到原来的逻辑，可以这样：
			    	a.Robot.Work() // 先按老流程走一遍，再加点自己的处理。
			  	fmt.Println("AdvancedRobot: I am working fast!")
			  }
			  
			  func main() {
			  	r := AdvancedRobot{}
			  
			  	// 调用的是 AdvancedRobot 自己的方法
			  	r.Work()
			  	// 输出: AdvancedRobot: I am working fast!
			  
			  	// 原始的方法依然存在，只是需要通过显式路径访问
			  	r.Robot.Work()
			  	// 输出: Robot: I am working...
			  }
			  
			  ```
	- **在结构体中嵌入接口**
	  collapsed:: true
		- 当你将一个接口类型直接声明在结构体中（不指定字段名，只写接口名）时，这就叫“嵌入接口”。
		- ```go
		  type Processor interface {
		      Process(data string) error
		  }
		  
		  type MyStruct struct {
		      // 这里嵌入了 Processor 接口
		      Processor
		  }
		  ```
		- **发生了什么？**
			- **字段隐式命名：**`MyStruct` 自动拥有了一个名为 `Processor` 的字段。
			- **方法提升：**`Processor` 接口中的所有方法都会被“提升”到 `MyStruct` 上。这意味着你可以直接在 `MyStruct` 的实例上调用 `Process()` 方法，而不需要写 `myStruct.Processor.Process()`。
			- **自动代理：**当你调用 `myStruct.Process()` 时，Go 实际上是把它转发（代理）给了内部存储的那个实现了 `Processor` 接口的具体对象。
		- **陷阱一：空指针 Panic**
			- 必须确保在调用方法前，内部的接口字段指向了一个具体的实现。
			- 嵌入接口本质上是一个字段。如果你初始化结构体时没有给这个接口字段赋值，它的默认值是 `nil`。
			- ```go
			  type MyStruct struct {
			      Processor // 接口
			  }
			  
			  func main() {
			      m := MyStruct{} // Processor 是 nil
			      m.Process("hello") //运行时 Panic！空指针引用
			  }
			  ```
	- **有什么用？**
	  collapsed:: true
		- **代码复用：**你不需要在每个结构体里重复写相同的字段。
		  logseq.order-list-type:: number
			- 在使用 GORM 时，几乎所有模型都会嵌入 `gorm.Model`，从而复用 `ID`，`CreatedAt`，`UpdatedAt`，`DeletedAt` 字段。
		- **快速满足接口：**你可以嵌入一个接口或结构体，通过“提升”机制，让外部结构体自动满足某个接口。
		  logseq.order-list-type:: number
- **结构体标签**
  collapsed:: true
	- 结构体标签的本质是字符串字面量，是一种附加在结构体字段后的“注解”或“元数据”。它不会影响程序逻辑，而是为反射机制和其他第三方库准备的元数据。
	- 结构体标签在 Go 中的作用类似于 Java 的注解，向其他程序包提供额外的元数据描述。
	- **标签的基本格式为：**`key:"value"`
	- **结构体标签种类及其作用：**
		- `json:"..."`：被 `encoding/json` 库读取，用于控制结构体与 JSON 之间的字段映射。
		- `gorm:"..."`：被 ORM 库（如 GORM）读取，用于定义数据库列名、主键、索引、约束等。
		- `db:"..."`：被 `database/sql` 库读取，用于定义字段与数据库列名的映射。
		- `validate:"..."`：被数据校验库读取，用于定义字段的验证规则。
	- **原理：**
		- 结构体标签的工作原理完全依赖于 Go 的反射包。
		- 当一个库（如 `encoding/json`）处理一个结构体时，它会执行以下步骤：
			- 使用 `reflect.TypeOf()` 获取结构体的类型信息。
			  logseq.order-list-type:: number
			- 遍历结构体的所有字段。
			  logseq.order-list-type:: number
			- 对每个字段，调用 `reflect.StructTag.Get("key")` 方法来提取标签字符串中特定键对应的值。
			  logseq.order-list-type:: number
			- 根据这个值来指导库的行为。
			  logseq.order-list-type:: number
	- **例如：**
		- ```go
		  type DownloadTask struct {
		  	ID         uuid.UUID `json:"id"`
		  	URL        string    `json:"url"`
		  	OutputPath string    `json:"output_path"`
		  	Threads    int       `json:"threads"`
		  }
		  ```
		- 这里的 `json:"threads"` 告诉 `encoding/json` 库：
			- 在序列化（Go → JSON）时，将字段 `Threads` 输出为 JSON 字段 `"threads"`；
			- 在反序列化（JSON → Go）时，当遇到键名 `"threads"` 时，将其值赋给结构体中的 `Threads` 字段。
		- 这样就能解决 Go 与 JSON 命名风格不同的问题：
			- Go 的导出字段要求首字母大写；
			- JSON 通常使用小写的 `camelCase` 或 `snake_case`。
			- 标签相当于两种命名规范之间的“翻译桥梁”。
- **Getter 和 Setter 方法**
  collapsed:: true
	- **Setter 方法**
		- 用于设置结构体字段的值，通常会在方法中加入合法性校验，避免设置无效或非法数据。
		- 封装是一种将数据隐藏、限制外部直接访问的机制，用于数据校验和状态保护。
		- Setter 方法必须使用指针接收者，否则只是修改副本，对原结构体没有影响。
		- 为防止外部绕过 Setter 直接修改字段，可以将字段设为未导出（小写），再提供一个导出（大写）的 Setter 方法来控制访问。
		- Go 的包级访问控制机制允许包内的导出函数访问未导出字段，因此可以通过导出方法间接访问未导出数据。
		- 按照惯例，Setter 方法通常以 `Set` 为前缀，例如 `SetAge`、`SetName`。
	- **Getter 方法**
		- 用于获取结构体字段或变量的值。
		- Go 通常不强制使用 Getter 方法，如果字段可以安全公开，直接导出字段即可。
		- 若需要通过方法访问，一般惯例是将方法命名为字段名本身，而不是使用 `Get` 前缀，例如访问字段 `Age` 的方法也应命名为 `Age()`，而不是 `GetAge()`，以保持简洁一致。
- **结构体的零值**
  collapsed:: true
	- 结构体是值类型，它的零值是其所有字段的零值集合 `{Age: 0}`，而不是 `nil`。
- **将结构体的成员字段指定为指针类型的原因：**
  collapsed:: true
	- **性能优化：避免复制大型结构体**
	  logseq.order-list-type:: number
		- 如果一个结构体（`StructA`）很大（包含许多字段或大型数组/切片），而您在另一个结构体（`StructB`）中需要引用它，有两种方式：
			- **值拷贝（非指针）：**
				- ```go
				  type LargeData struct { /* ... 很多字段 ... */ }
				  type Container struct {
				      Data LargeData // 拷贝整个 LargeData
				  }
				  ```
				- 当创建 `Container` 实例时，整个 `LargeData` 会被复制到 `Container` 的内存空间中。这会消耗**更多的内存和 CPU 时间**。
			- **指针引用：**
				- ```go
				  type LargeData struct { /* ... 很多字段 ... */ }
				  type Container struct {
				      DataPtr *LargeData // 只存储一个指针（地址）
				  }
				  ```
				- 当使用指针时，`Container` 只需要存储一个固定大小的**内存地址**（通常是 8 字节）。**只有地址被复制**，而不是整个大型数据结构。这显著提高了创建和传递结构体实例的**性能**。
	- **实现“可选”或“空值”语义**
	  logseq.order-list-type:: number
		- **Go 语言类型的“零值”特性**
			- Go 语言的设计哲学之一是默认值是安全的。这意味着任何变量一旦声明，即使没有显式赋值，它也会被赋予其类型的“零值”：
				- `int` 的零值是 `0`。
				- `bool` 的零值是 `false`。
				- `string` 的零值是 `""`（空字符串）。
				- 结构体（Struct）的零值是所有字段的零值组合。
			- **问题所在：**对于像 `Age` 这样的数值类型，`0` 可能是一个**有效值**（比如一个人是 0 岁，或者在数据记录中年龄为 0 表示“未记录”但系统需要一个默认值）。但是，如果业务需求是区分“年龄是 0 岁”和“年龄信息缺失/未知”，那么仅靠 `int` 类型的零值是做不到的。
		- **指针如何解决“零值歧义”**
			- 当我们将字段类型改为指针 `*int` 时：
				- **零值 (Zero Value)：** `*int` 类型的零值是 **`nil`**。
				- **有效值 (Valid Value)：** 任何指向一个实际 `int` 变量的地址。
			- `nil` 表示：信息缺失/可选。非 `nil` 表示：信息已提供。
-