- **为什么要自定义错误？**
	- 有时候，像 `errors.New("出错了")` 这样简单的错误信息字符串是不够的。我们更希望错误本身能携带更丰富的信息——比如，当函数因参数无效而失败时，除了“参数无效”这一结论，我们还想明确知道具体是哪个参数出了问题。
	- 自定义错误正是为解决这一问题而生：通过创建自定义 `struct` 来承载这些详细信息，并使其实现 `error` 接口，从而让它表现得像一个 `error`。
- **第一步：定义自定义错误结构体**
	- ```go
	  type argError struct {
	      arg     int
	      message string
	  }
	  ```
	- **命名约定**：Go 社区通常建议自定义错误的类型名称以后缀 `Error` 结尾。
	- 这个结构体有两个字段：
		- `arg int`：用来存放导致错误的那个具体参数值。
		- `message string`：用来存放错误的文本描述。
	- 通过这个结构体，我们就把“哪个参数”和“什么原因”这两个信息绑定在了一起。
- **第二步：实现  `error`  接口**
	- 为了让 Go 语言认可我们的 `argError` 是一种 `error`，它必须满足 `error` 接口。
	- `error` 接口非常简单，只要求实现一个方法：
		- ```go
		  type error interface {
		      Error() string
		  }
		  ```
	- 所以，我们为 `argError` 添加这个 `Error()` 方法：
		- ```go
		  func (e *argError) Error() string {
		      return fmt.Sprintf("%d - %s", e.arg, e.message)
		  }
		  ```
		- 注意这里用的是指针接收者 `(e *argError)`。这意味着 `*argError` 类型实现了 `error` 接口。这是 Go 中的常见做法。
		- 这个方法定义了当这个错误被当成一个普通字符串打印时（例如 `fmt.Println(err)`），应该显示什么内容。这里它格式化输出了参数值和错误信息。
	- 完成这一步后，`*argError` 现在就是一个合法的 `error` 类型了。
- **第三步：在函数中使用自定义错误**
	- 现在我们可以在函数中创建并返回这个自定义错误了：
		- ```go
		  func f(arg int) (int, error) {
		      if arg == 42 {
		          // 返回我们自定义的错误类型
		          // 注意我们返回的是一个指向结构体实例的指针
		          return -1, &argError{arg, "can't work with it"}
		      }
		      return arg + 3, nil
		  }
		  ```
		- 当 `arg` 的值是 `42` 时，函数不再返回一个普通的 `error`，而是创建了一个 `argError` 的实例，并把导致错误的参数 `arg` (也就是42) 和错误信息 `"can't work with it"` 存了进去。
		- 它返回的是 `&argError{...}`，一个指向该实例的指针，这与我们之前定义的指针接收者 `(e *argError)` 相匹配。
- **第四步：处理并解析自定义错误**
	- ```go
	  func main() {
	      _, err := f(42) // 调用函数，接收到我们的自定义错误
	      
	      // 声明一个目标变量，用于接收解析出来的自定义错误
	      var ae *argError
	  
	      // 使用 errors.As 来检查并转换错误
	      if errors.As(err, &ae) {
	          // 如果 `err` 链中确实包含一个 `*argError` 类型的错误
	          // `errors.As` 会返回 true，并将那个错误赋值给 ae
	          
	          // 现在我们可以访问 ae 中的特定字段了！
	          fmt.Println(ae.arg)
	          fmt.Println(ae.message)
	      } else {
	          // 如果返回的错误不是 *argError 类型
	          fmt.Println("err doesn't match argError")
	      }
	  }
	  ```
	- 我们首先调用 `f(42)`，`err` 变量现在持有的就是我们返回的 `*argError`。
	  logseq.order-list-type:: number
	- 然后我们用 `var ae *argError` 声明了一个 `ae` 变量。它的类型是我们期望从 `err` 中提取出的具体错误类型。
	  logseq.order-list-type:: number
	- `errors.As(err, &ae)` 这行代码做了两件事：
	  logseq.order-list-type:: number
		- 检查 `err` 是不是 `*argError` 类型。
		  logseq.order-list-type:: number
		- 如果是，就把 `err` 的值赋给 `ae`，并返回 `true`。
		  logseq.order-list-type:: number
	- 因为 `errors.As` 返回 `true`，代码块被执行。此时 `ae` 变量已经指向了那个 `argError` 实例，所以我们可以通过 `ae.arg` 和 `ae.message` 直接访问里面的结构化数据。
	  logseq.order-list-type:: number
- **整个流程可以总结为：**
	- **定义**：创建一个 `struct` 来存放你需要的错误上下文信息。
	- **实现**：为你的 `struct` 实现 `Error() string` 方法，让它满足 `error` 接口。
	- **返回**：在你的函数中，当特定条件满足时，创建并返回你的自定义错误实例。
	- **解析**：在调用方，使用 `errors.As()` 来安全地检查错误的类型，并在成功后获取其实例，从而访问你预先定义好的结构化数据。
-