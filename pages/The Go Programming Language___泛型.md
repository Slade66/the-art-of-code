- 类型参数（泛型）是 Go 语言实现多态性的一种方式，其核心目的是：
	- **代码复用：**编写一次函数，使其可以安全地用于多种不同的数据类型。
	- **类型安全：**在编译时检查类型，而不是在运行时引发错误。
- **语法结构：**
	- 类型参数被定义在函数名和常规参数列表之间的一对方括号 `[]` 内。
	- ```go
	  func Index[T comparable](s []T, x T) int
	  //         ^^^^^^^^^^^
	  //         类型参数列表
	  ```
	- **`[T comparable]`：**声明了一个类型参数 `T`。
		- `T`：这是类型参数的名称，惯例上使用单个大写字母。
		- `comparable`：这是对类型参数 `T` 的约束。
- **类型约束：**
	- 在泛型编程中，我们不能对任意类型执行任意操作，比如并非所有类型都能进行加法。约束的作用就是通过接口（约束的本质）告诉编译器，类型参数需要具备哪些能力，即可以执行哪些操作。限制类型参数的传参范围。
	- 把约束想成“准入名单”或“能力清单”。当你写 `type X interface { A | B | C }`，就是在告诉编译器：只有名单上这些类型（或实现了这些接口的具体类型）才被允许进入这个泛型函数/类型。
	- **编译时验证：**
		- 当您调用一个泛型函数时，Go 编译器会尝试推断出 T 的具体类型，编译器会检查函数体内部对 T 执行的所有操作是否都已在 T 的约束中被允许。
		- ```go
		  func Add[T constraints.Integer](a, b T) T {
		      return a + b // 编译器检查 T 是否是 Integer（即支持 +）
		  }
		  
		  // 场景 1: 类型安全
		  Add(10, 20) // T 被推断为 int，编译器验证通过。
		  
		  // 场景 2: 编译时报错 (类型不安全)
		  Add("hello", "world") // 编译器报错：string 不满足 constraints.Integer 约束。
		  
		  // 空接口的风险示例：
		  
		  func UnsafeAdd(a, b interface{}) interface{} {
		      // 必须进行类型断言，如果有人传入了字符串，这里会 panic
		      sum := a.(int) + b.(int) // <-- 风险点！
		      return sum
		  }
		  
		  UnsafeAdd(10, 20) // 成功
		  UnsafeAdd(10, "wrong") // 运行时 panic: interface conversion: string does not match int
		  ```
			- 如果 `Add` 函数没有泛型，并且您使用了传统的空接口，那么 `Add("hello", "world")` 可能会被允许传入，但在函数内部尝试执行 `"hello" + "world"`（字符串拼接），而不是整数相加，这可能是**逻辑错误**，或者如果内部尝试强制类型转换为数字，则会导致**运行时 panic**。
			- 使用泛型，如上文的 `Add[T constraints.Integer](a, b T)`，编译器会阻止你将字符串传入 `UnsafeAdd` 这种泛型版本，从而消除了运行时 `panic` 的风险。
			- 泛型通过约束机制，明确定义了类型参数可以做什么，并将所有不符合这些定义的调用都拦截在编译阶段。
	- **内置约束：**
		- `comparable`：必须支持 `==` 和 `!=` 运算符。
		- `any`：等同于 `interface{}`，即不施加任何限制。任何类型都可以替代。
		- `ordered`：必须支持所有顺序比较运算符 (`<`, `<=`, `>`, `>=`)。
	- **自定义约束：**
		- **接口的联合：**
			- ```go
			  type Number interface {
			      int | int8 | int16 | int32 | int64 |
			      uint | uint8 | uint16 | uint32 | uint64 |
			      float32 | float64
			  }
			  ```
			- 使用 `|` 符号在接口中列出所有被允许的类型，只要是名单里任意一项，都能满足该约束。
		- **定义方法约束：**
			- ```go
			  type Stringer interface {
			      String() string
			  }
			  
			  // IndexStringer 期望 T 类型有一个 String() string 方法
			  func IndexStringer[T Stringer](items []T, x T) int {
			      // 可以在这里使用 x.String() 或 items[i].String()
			      // ...
			      return -1
			  }
			  ```
			- 在这个例子中，任何传入的类型 `T` 都必须拥有一个不带参数、返回值为 `string` 的 `String()` 方法。
- **示例：**
	- 一个通用的 `Index` 函数：
		- ```go
		  func Index[T comparable](s []T, x T) int {
		      for i, v := range s {
		          // v and x are type T, which has the comparable
		          // constraint, so we can use == here.
		          if v == x {
		              return i
		          }
		      }
		      return -1
		  }
		  ```
	- **函数签名：**
		- `[T comparable]`：定义了一个受 `comparable` 约束的类型参数 `T`。
		- `s []T`：第一个参数 `s` 必须是一个切片，其元素类型是 `T`。
		- `x T`：第二个参数 `x` 是要查找的值，其类型也是 `T`。
		- `int`：返回值是一个整数，表示找到的索引或未找到时的 `-1`。
	- **函数逻辑：**
		- 函数通过 `for i, v := range s` 遍历切片 `s`。
		- 在第 10 行，执行了关键的比较操作 `if v == x`。
		- **类型安全保障：**编译器在编译时看到 `T` 有 `comparable` 约束，就知道这个 `==` 操作是合法的。如果没有这个约束，编译器将报告错误。
		- 如果找到匹配项，返回索引 `i`；遍历结束未找到，则返回 `-1`。
	- **使用演示：**
		- **用于 `int` 切片：**
			- ```go
			  // Index works on a slice of ints
			  si := []int{10, 20, 15, -10}
			  fmt.Println(Index(si, 15)) // 输出: 2
			  ```
			- 在这个调用中，Go 编译器会进行类型推断：
				- 它根据切片 `si` 的类型 `[]int`，推断出类型参数 `T` 为 `int`。
				- 由于 `int` 类型是 `comparable` 的，所以调用成功。
		- **用于 `string` 切片：**
			- 编译器根据切片 `ss` 的类型 `[]string`，推断出类型参数 `T` 为 `string`。
			- 由于 `string` 类型也是 `comparable` 的，所以代码可以复用，且类型安全。
	- 如果没有泛型，您需要为 `int` 编写一个 `IndexInt` 函数，为 `string` 编写一个 `IndexString` 函数，或使用低效且不安全的空接口 (`interface{}`)。泛型（类型参数）解决了代码复用的问题，实现了高效、类型安全的通用算法。
-