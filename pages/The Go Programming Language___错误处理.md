- **Go 语言的错误处理方式：**
  collapsed:: true
	- Go 语言在设计上独树一帜，它摒弃了许多主流语言中常见的异常（exception）机制。
	- **错误即是值：**错误被视为函数的普通返回值，与任何其他数据类型（如整数或字符串）具有同等地位。通常将 `error` 类型的值作为可能失败的函数的最后一个返回值，用于表示可能发生的错误。
	- Go 强制程序员在每个潜在的故障点进行有意识的决策，通过将错误作为显式的返回值，Go 强制开发者在每个可能出错的地方直面失败的可能性，这种明确性是构建健壮、可维护系统的关键，从而构建出更可靠的软件。
	- 每个可能出错的函数通常会返回至少两个值：一个是期望的结果，另一个是 `error` 类型的错误。
		- `error` 为 `nil`：表示操作成功，没有发生错误。
		  id:: 68f6fc86-a943-4aa2-acf5-90623686ed88
		- `error` 非 `nil`：表示操作失败，出现了错误。
		  id:: 68f6fc8a-603f-49c7-b337-6b1076ea78f7
	- **示例：**
		- ```go
		  // 尝试做某事，它可能成功（返回真实值, nil）
		  // 或者失败（返回零值, error）
		  value, err := doSomething()
		  if err != nil {
		      // 失败了！
		      // 立即处理这个错误（比如：打日志、返回、或panic）
		      return err
		  }
		  // 成功了！
		  // ...继续使用 value...
		  ```
- **异常机制的弊端：**
  collapsed:: true
	- **栈解旋（Stack Unwinding）的成本：**
		- 基于异常的错误处理在“成功路径”（即没有错误发生时）上通常性能开销很小。但在异常被抛出时，会触发一个名为“栈解旋”的复杂且昂贵的过程。
		- 每个函数对应一个“栈帧”（存局部变量、返回地址、defer 等）。
		- 当程序执行到某个函数内部发生异常时，运行时会从当前函数（栈顶）开始，一层层往上回溯调用栈，逐层弹出栈帧，查找异常处理器，执行每层的清理代码（defer / finally / 析构函数）并销毁局部变量，如果没有 recover，最终整个栈都被弹空，程序退出。
		- 这个过程可能比一次简单的函数返回慢上百倍甚至千倍。
		- **执行过程：**
			- `main → A → B → C (panic)`
			- panic 发生后运行时做的事：
				- 执行 C 的 defer → 销毁 C
				  logseq.order-list-type:: number
				- 执行 B 的 defer → 销毁 B
				  logseq.order-list-type:: number
				- 执行 A 的 defer → 销毁 A
				  logseq.order-list-type:: number
				- 到 main，被 recover 捕获（或程序崩溃）
				  logseq.order-list-type:: number
- **`panic` 函数：**
  collapsed:: true
	- `panic` 是 Go 的一个内置函数，它会中断程序的正常执行流程。你可以把它想象成一颗“手榴弹”💣，一旦拉响（被调用），它就会立即引爆当前的任务（goroutine），除非被特殊机制“拆除”（`recover`）。
	- 当函数调用 `panic` 时，会依次触发以下事件：
		- **立即终止执行**：当前函数的执行会即刻停止，后续代码不再运行。
		- **执行 `defer` 语句**：
			- 该函数中所有被延迟（`defer`）的调用会按后进先出的顺序立即执行。这为资源清理（如关闭文件、释放锁）提供了一个机会。
			- 这是 `panic` 与 `os.Exit` 的核心区别 —— 后者会直接终止程序，不会执行 `defer`。
		- **沿调用栈向上“冒泡”**：`panic` 状态会传递给调用者，重复执行“中断”和“执行 Defer”的过程，并沿着当前 goroutine 的调用栈逐层向上蔓延。
		- **程序崩溃**：若 “恐慌” 状态蔓延至 goroutine 调用栈顶端仍未被处理，程序就会崩溃，此时会打印 `panic` 传入的值和完整栈追踪信息，并以非零状态码退出。
	- **示例：`panic` 的“冒泡”之旅**
		- 当一个函数中发生了 `panic`，正常的“函数返回”流程就被打断了，取而代之的是“栈解退”的冒泡过程。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func functionB() {
		      // defer 在 panic "冒泡"离开本函数前执行
		      defer fmt.Println("清理 functionB...")
		      
		      fmt.Println("进入 functionB")
		      panic("发生了一个重大问题！") // panic 被触发！
		      
		      // panic 下面的代码永远不会被执行
		      fmt.Println("离开 functionB") 
		  }
		  
		  func functionA() {
		      defer fmt.Println("清理 functionA...")
		      
		      fmt.Println("进入 functionA")
		      functionB() // 调用 functionB
		      fmt.Println("离开 functionA")
		  }
		  
		  func main() {
		      defer fmt.Println("清理 main...")
		      
		      fmt.Println("进入 main")
		      functionA() // 调用 functionA
		      fmt.Println("离开 main")
		  }
		  
		  // 程序的最终输出会是：
		  进入 main
		  进入 functionA
		  进入 functionB
		  清理 functionB...
		  清理 functionA...
		  清理 main...
		  panic: 发生了一个重大问题！
		  
		  goroutine 1 [running]:
		  main.functionB()
		  	/path/to/your/program.go:12 +0xbf
		  main.functionA()
		  	/path/to/your/program.go:21 +0x7d
		  main.main()
		  	/path/to/your/program.go:28 +0x7d
		  ... (其他信息)
		  ```
	- **何时应该使用 `panic`？**
		- 始终避免使用 `panic` 处理普通错误，这类错误应通过返回 `error` 来处理，而 `panic` 仅用于真正的异常情况。
		- `panic` 的合理使用场景仅限于程序遇到不可恢复的严重错误，导致继续运行无意义时，此时应使用它 “快速失败”。在这些情况下，让程序快速失败并打印出详细的堆栈跟踪信息，比让其在一种未知的、可能已损坏的状态下继续运行要安全得多。
		- 典型场景包括关键配置文件加载失败、必要依赖服务不可用等。
- **可恢复错误 vs. 不可恢复错误**
  collapsed:: true
	- **`error`**：用于表示**可预期的、可恢复的**失败。这些是程序正常运行过程中可能发生的情况，例如文件未找到、网络请求超时、用户输入格式错误等 。`error` 是函数签名的一部分，它明确告知调用者该函数可能会失败，并期望调用者对其进行处理。
	- **`panic`**：用于表示**异常的、通常是不可恢复的**运行时错误。`panic` 的出现通常意味着程序进入了一个意料之外的、严重损坏的状态，以至于无法安全地继续执行。典型的例子包括空指针解引用、数组越界访问、并发访问 map 时的竞态条件等。这些情况往往表明程序中存在一个缺陷。
- **`recover` 函数：**
  collapsed:: true
	- `panic` 的冒泡行为是可以阻止的。`recover()` 函数可以在 `defer` 中捕获一个正在“冒泡”的 `panic`，从而“戳破”这个气泡，使程序从恐慌状态中恢复，避免崩溃并继续运行。
	- **`recover()` 必须在 `defer` 语句中调用才能生效。**
		- 为什么？因为当 `panic` 发生时，正常的代码执行流程会立即停止，只有 `defer` 中的任务会继续执行。因此，`defer` 函数是唯一能在 `panic` 发生后、程序崩溃前进行干预的地方。
	- 如果在没有 `panic` 的情况下调用 `recover()`，或在 `defer` 之外调用它，它不会起任何作用，并会返回 `nil`。
	- 如果成功捕获 `panic`，`recover` 会返回传给 `panic` 的值。
	- 捕获了 `panic` 的函数，自己也无法从发生 `panic` 的那个调用点继续向后执行，只能从那返回。
	- **`recover` 的主要应用场景：**
		- `recover` 的主要惯用场景是在一个长期运行的应用程序（如 Web 服务器）中充当“安全网”。
		- 一个典型的例子是在 HTTP 服务器的中间件中使用 `recover`。
		- 如果某个具体的请求处理器（handler）因为一个未预料到的 bug 而发生了 `panic`，这个中间件可以捕获该 `panic`，记录详细的错误日志，向客户端返回一个 `500 Internal Server Error` 响应，然后让服务器继续处理其他正常的请求。
		- 这样，单个请求的失败就不会导致整个服务进程崩溃，从而提高了服务的整体健壮性。
		- ```go
		  func main() {
		      http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		          defer func() {
		              if r := recover(); r!= nil {
		                  log.Printf("panic recovered: %v", r)
		                  http.Error(w, "Internal Server Error", http.StatusInternalServerError)
		              }
		          }()
		  
		          // 可能会发生 panic 的业务逻辑
		          //...
		      })
		      http.ListenAndServe(":8080", nil)
		  }
		  ```
	- **示例：**
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		  	fmt.Println("main 开始")
		  	f1()
		  	fmt.Println("main 结束")
		  }
		  
		  func f1() {
		  	// 用 defer 包裹 recover
		  	defer func() {
		  		if err := recover(); err != nil {
		  			fmt.Println("捕获到 panic：", err)
		  		}
		  	}()
		  	fmt.Println("f1 开始")
		  	f2()
		  	fmt.Println("f1 结束")
		  }
		  
		  func f2() {
		  	fmt.Println("f2 开始")
		  	panic("出了点问题！")
		  	fmt.Println("f2 结束")
		  }
		  
		  ```
- [[type error interface]]
- **自定义错误类型：**
  collapsed:: true
	- **步骤：**
		- **第 1 步：自定义类型（装数据）**
			- 创建一个你自己的新类型（通常是一个 `struct`，`int` 或 `float64` 等也可以），用来作为“容器”装载你希望错误携带的任何上下文数据（比如 `UserID`、`ErrorCode` 等）。
			- 通过定义一个实现了 `error` 接口的 `struct`，我们可以创建出能够携带丰富上下文信息的错误 。
		- **第 2 步：实现 `error` 接口的方法（变 `error`）**
			- 一个类型能被当作 `error` 使用的唯一条件是：它必须实现 `Error() string` 方法。
			- 为你的新类型添加一个名为 `Error() string` 的方法。这个方法必须返回一个字符串，用来描述这个错误“错在了哪里”。
			- 一旦你做了这一步，Go 就正式承认你的类型是一个合法的 `error` 了。
		- **第 3 步：返回实例（用起来）**
			- 在你的函数中，当特定错误发生时，创建这个新类型的实例（比如 `&MyError{UserID: 404}`），并把它作为 `error` 类型返回。
	- **示例代码：**
		- **自定义结构体类型的错误：**
			- ```go
			  package main
			  
			  import "fmt"
			  
			  type ValidationError struct {
			      Field   string
			      Message string
			  }
			  
			  func (e *ValidationError) Error() string {
			      return fmt.Sprintf("validation failed on '%s': %s", e.Field, e.Message)
			  }
			  
			  func validateEmail(email string) error {
			      if email == "" {
			          return &ValidationError{Field: "email", Message: "cannot be empty"}
			      }
			      //... 更多验证逻辑
			      return nil
			  }
			  ```
		- **给错误类型加上状态码：**
			- 在真实世界的应用中，错误通常需要携带机器可读的数据（内部错误码），以便程序能够根据错误内容做出自动化决策。这种模式常用于对内部错误进行分类，便于监控和告警。
			- ```go
			  type AppError struct {
			      Code    int
			      Message string
			  }
			  
			  func (e *AppError) Error() string {
			      return fmt.Sprintf("code %d: %s", e.Code, e.Message)
			  }
			  
			  const (
			      ErrCodeInvalidInput = 1001
			      ErrCodeNotFound     = 1002
			  )
			  ```
		- **为自定义类型实现 `Unwrap`：**
			- 为了让自定义错误类型能够完全融入 Go 1.13+ 的错误处理生态，如果它内部包装了另一个错误，就应该为其实现 `Unwrap` 方法。
			- 通过实现 `Unwrap`，我们允许 `errors.Is` 和 `errors.As` 函数“看穿”我们的自定义错误，检查其内部包装的错误链。如果没有这个方法，错误链在我们的自定义类型处就会中断，导致上层调用者无法检查到底层原因。
			- ```go
			  type RequestError struct {
			      StatusCode int
			      Err        error // 底层错误
			  }
			  
			  func (r *RequestError) Error() string {
			      return fmt.Sprintf("status %d: %v", r.StatusCode, r.Err)
			  }
			  
			  // 实现 Unwrap 方法，直接返回底层的错误
			  func (r *RequestError) Unwrap() error {
			      return r.Err
			  }
			  ```
- [[The Go Programming Language/Standard library/errors]]
- ((68fdc145-4dba-4b18-83b1-ffaa52ce82a2))
- **哨兵错误（Sentinel Error）**
  collapsed:: true
	- 哨兵错误是指在包级别定义的、可导出的错误变量，用于表示特定的错误。
	- **示例：**
		- ```go
		  var EOF = errors.New("EOF") // 它表示“输入流已经读到末尾”，即再也没有数据可读了。
		  
		  // 调用方这样用：
		  
		  package main
		  
		  import (
		  	"fmt"
		  	"io"
		  	"strings"
		  )
		  
		  func main() {
		  	r := strings.NewReader("abc")
		  	buf := make([]byte, 2)
		  
		  	for {
		  		n, err := r.Read(buf)
		  		if err != nil {
		  			if err == io.EOF {
		  				fmt.Println("读完了")
		  				break
		  			}
		  			fmt.Println("发生错误：", err)
		  			break
		  		}
		  		fmt.Printf("读取到 %d 字节：%s\n", n, string(buf[:n]))
		  	}
		  }
		  
		  ```
	- **为什么可以这样用？**
		- `errors.New("EOF")` 返回的是一个固定的 `error` 对象（在内存中有唯一地址）。
		- 所以当标准库返回 `io.EOF` 时，调用方拿到的 `err` 实际上就是同一个对象的引用，因此调用方可以直接用 `==` 来判断，比较的是同一个对象。
		- 这种写法成立的前提是：包内始终返回的就是同一个变量 `io.EOF`，而不是一个新建的错误对象。
		- 如果标准库每次都用 `errors.New("EOF")` 新建一个错误，那么即使错误信息一样，`==` 也会判断为 `false`，因为是不同对象。
	- **哨兵错误的隐患与缺点：**
		- **包耦合（依赖关系）变强：**
			- 为了检查 ErrUserNotFound，调用者必须导入 user 包。大量使用哨兵错误会让很多包相互依赖，甚至可能引发循环导入问题。
		- **限制实现演进（向后兼容问题）：**
			- 哨兵错误通常只是一个字符串，不能携带额外信息（比如哪个用户 ID 未找到）。这使得调用方无法根据更多上下文作更细粒度的决策。
			- 一旦一个函数承诺返回某个特定的哨兵错误值，它就很难在未来改变其实现以返回一个包含更多上下文信息的、更丰富的错误类型（例如一个自定义结构体），因为这样做会破坏那些依赖 `err == pkg.ErrSomething` 检查的客户端代码。
- ## 错误处理的原则
  collapsed:: true
	- ### Don't log and return（不要“记录日志后再返回错误”）
		- 初学者常犯的一个错误是：在函数内部既记录错误日志，又将该错误返回。这样做会导致同一个错误在调用栈的每一层都被重复记录，生成大量冗余、嘈杂的日志，反而使问题的根源更难追踪。
		- ```go
		  // 反例：不要这样做
		  func createUser(name string) error {
		      if name == "" {
		          err := errors.New("user name cannot be empty")
		          log.Printf("ERROR: failed to create user: %v", err) // 在这里记录日志
		          return err // 然后又返回错误
		      }
		      // ...
		      return nil
		  }
		  
		  ```
		- **正确的做法是：**
			- 底层函数（如库或通用组件）只负责报告失败，即返回错误，不打印日志。
			- 高层调用者（业务逻辑层）才负责决策——例如是重试、忽略、还是记录日志。
			- 一个错误只应被“处理”一次，而记录日志本身就是一种处理方式。最好的做法是将错误向上传递，让最了解上下文的调用者决定该如何应对。
	- ### 底层使用错误包装 ( `%w` )
		- 在返回错误时，使用 `fmt.Errorf("上下文信息: %w", originalError)`，既保留**原始错误（根因）**，又补充当前函数的上下文信息。上层可通过 `errors.Is()` 或 `errors.As()` 判断原始错误类型。
	- ### 返回优先
		- 函数只有在**具备足够上下文**、能够**采取有意义的措施**（例如重试、使用默认值或后备方案）时，才应主动处理错误。
		- **否则**，应使用 `%w` 包装错误，添加必要的上下文后**直接返回**，将处理责任交给上层调用者。
-