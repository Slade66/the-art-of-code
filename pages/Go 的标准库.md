`fmt`
heading:: true
	- **作用：**提供格式化的输入输出功能，用于将信息打印到控制台或读取用户输入。
	- **特点：**
		- `fmt` 包通常不添加时间戳，直接输出格式化的信息。
	- 格式化的语法结构：
		- ```go
		  %[flags][width][.precision][verb]
		  ```
		- **width（宽度）**：指定输出所占的最小字符数。不足部分会用空格填充。正数表示右对齐，负数表示左对齐。
		- **precision（精度）**：含义因类型而异。对于浮点数，表示小数点后的位数；如果实际位数不足，会自动补零，超过则进行四舍五入。
		- 格式化占位符：
			- `%v`：可以打印任意类型的值，无需手动指定格式，输出方式会根据变量类型自动选择最合适的表现形式。
			- `%#v`：会将值按照 Go 源码的语法格式打印出来
			- `%T`：输出变量的类型
			- `%t`：输出布尔型
			- `%%`：输出 `%` 字符
			- `%s`：输出字符串
			- `%q`：输出双引号包裹的字符串
			- `%c`：输出数值对应的字符
			- `%d`：十进制整数输出
			- `%b`：二进制整数输出
			- `%o`：八进制整数输出
			- `%x`：十六进制整数（小写）
			- `%X`：十六进制整数（大写）
			- `%f`：浮点数（默认小数点后 6 位）
			- `%e`：科学计数法（小写 e）
			- `%E`：科学计数法（大写 E）
	- `func Println(a ...any) (n int, err error)`
		- 用于将参数打印到终端。
		- 参数之间会自动加空格，末尾会自动添加一个换行符。
		- 打印数组和切片时，会打印出它们的所有元素，并用空格分隔，左右用方括号括起来。
	- `func Printf(format string, a ...any) (n int, err error)`
		- `Printf` 是一个格式化输出函数。它接收一个格式化字符串 `format` 和任意数量的参数 `a`（可变参数 `...any`），并按照指定的格式将它们输出到标准输出。
		- 返回值 `n` 表示实际输出的字符数，`err` 表示输出过程中的任何错误（通常为 `nil`）。
	- `func Sprintf(format string, a ...any) string`
		- 类似于 `printf`，但返回字符串而不是直接输出。
	- `func Errorf(format string, a ...any) error`
		- **作用**：
			- 用格式化的方式创建一个错误值，方便插入变量信息。
			- 还可以嵌套另一个错误，把原始错误“包装”在新的错误信息中一并返回。
		- **返回值**：
			- 一个实现了 `error` 接口的对象，包含你格式化后的错误信息。
		- **示例：**
			- ```go
			  func readConfig() error {
			      base := errors.New("文件未找到")
			      return fmt.Errorf("读取配置失败：%w", base)
			  }
			  
			  func main() {
			      err := readConfig()
			      fmt.Println(err) // 输出：读取配置失败：文件未找到
			  }
			  ```
- `os`
  heading:: true
	- `var Args []string`
		- `os.Args` 是一个字符串切片，存储命令行参数。
		- `os.Args[0]` 保存的是程序自身的路径（可执行文件名）。
		- 实际的参数从 `os.Args[1:]` 开始。
	- `type File struct {...}`
		- `func Open(name string) (*File, error)`
			- **作用：**
				- 以只读方式打开一个文件，返回文件对象和可能的错误。
			- **参数：**
				- `name` 是要打开的文件名或路径（字符串类型），可以是相对路径或绝对路径。
			- **返回值：**
				- `*File`：打开的文件对象指针（类型为 `*os.File`）
				- `error`：如果打开失败（如文件不存在或权限不足），返回非 `nil` 的错误
			- **注意：**
				- 打开的文件是只读的，不能写入；如需写入，请使用 `os.OpenFile` 函数。
		- `func (f *File) Close() error`
			- **作用**：
				- 关闭文件，释放相关的系统资源（如文件描述符）。
			- **参数**：
				- 无参数。该方法是 `*File` 的接收者，调用者是一个打开的文件对象。
			- **返回值**：
				- 返回一个 `error` 类型的值。
					- 如果关闭成功，返回 `nil`；
					- 如果关闭失败，返回错误信息（例如设备异常、文件系统错误等）。
- `strings`
  heading:: true
	- 提供了用于字符串处理的函数。
	- `func Join(elems []string, sep string) string`
		- `Join` 会将一组字符串用指定的分隔符拼接成一个完整的字符串。简单来说，它就是把切片里的所有元素串在一起，中间插入分隔符 `sep`。
	- `func TrimSpace(s string) string`
		- **作用**：
			- 删除字符串开头和结尾处的所有空白字符（包括空格、换行符、制表符等）。
		- **参数**：
			- `s` 是要处理的字符串。
		- **返回值**：
			- 返回一个去除首尾空白字符后的新字符串（原字符串不变）。
		- **示例**：
			- ```go
			  str := " \t\n Hello, Go! \n\t "
			  result := strings.TrimSpace(str)
			  fmt.Println(result) // 输出：Hello, Go!
			  ```
	- `func NewReplacer(oldnew ...string) *Replacer`
		- **作用**：创建一个 `Replacer` 对象，用于定义一组字符串替换规则。
		- **参数**：可变参数 `oldnew`，格式必须是成对出现的 `"旧字符串", "新字符串"`。
		- **返回值**：一个指向 `Replacer` 的指针。
		- **示例**：
			- ```go
			  r := strings.NewReplacer("cat", "dog", "hello", "hi")
			  ```
			- 这个 `Replacer` 表示：
				- 替换 `"cat"` 为 `"dog"`
				- 替换 `"hello"` 为 `"hi"`
	- `func (r *Replacer) Replace(s string) string`
		- **作用**：对传入的字符串 `s` 按照 `Replacer` 中设定的规则进行批量替换，并返回替换后的结果。
		- **参数**：一个待处理的字符串 `s`
		- **返回值**：替换后的字符串（原字符串不变）
		- **示例**：
			- ```go
			  result := r.Replace("hello cat!")
			  fmt.Println(result) // 输出：hi dog!
			  ```
- `bufio`
  heading:: true
	- 在 I/O 操作时，使用缓冲区来减少系统调用次数，提升性能。
	- `type Scanner struct {...}`
		- 用于按行或自定义分隔符扫描输入数据的扫描器。
		- `func NewScanner(r io.Reader) *Scanner`
			- 创建一个新的扫描器，接收一个 `io.Reader`（例如文件、标准输入等），返回一个可以从中读取数据的 `Scanner` 实例。
		- `func (s *Scanner) Scan() bool`
			- 移动扫描器到下一行（或下一个分隔的片段），如果还有数据可读，返回 `true`；否则返回 `false`。
		- `func (s *Scanner) Text() string`
			- 返回最近一次调用 `Scan` 时扫描到的文本内容，通常是一行字符串。
	- `type Reader struct {...}`
		- Go 标准库中的 `bufio.Reader` 是一个带缓冲的读取器，它在你读取数据之前，会一次性从数据源中读入一大块内容并存入内存缓冲区，之后每次读取时直接从缓冲区中取出所需的数据，从而提升读取速度并减少资源消耗。
		- `func NewReader(rd io.Reader) *Reader`
			- **作用**：
				- 创建一个“带缓冲区的读取器”。
			- **参数**：
				- `rd` 是你要读取的来源，比如文件、网络连接、标准输入等。只要它实现了 `io.Reader` 接口（大多数可读对象都实现了），就可以使用。
			- **返回值**：
				- 返回一个 `*bufio.Reader` 对象，你可以用它来方便地读取数据，比如读一行、读一个字节、读到某个字符等。
			- **示例**：
				- ```go
				  file, _ := os.Open("example.txt")           // 打开一个文件
				  reader := bufio.NewReader(file)             // 用缓冲读取器包装它
				  line, _ := reader.ReadString('\n')          // 读取一行内容
				  fmt.Println("读取到一行：", line)           // 打印结果
				  ```
			- **注意：**
				- `NewReader` 会自动使用一个默认大小的缓冲区（通常是 4KB）。
		- `func (b *Reader) ReadString(delim byte) (string, error)`
			- **作用**：
				- 从输入中读取内容，直到遇到指定的分隔符 `delim` 为止，并返回包含该分隔符在内的字符串。
			- **参数**：
				- `delim` 是要读取到的“终止字符”，如换行符 `'\n'`、逗号 `','` 等。
			- **返回值**：
				- 返回读取到的字符串（包含分隔符本身）以及可能发生的错误。
			- **示例**：
				- ```go
				  reader := bufio.NewReader(strings.NewReader("hello,world\nGo!"))
				  line, err := reader.ReadString('\n') // 读取直到遇到换行符
				  fmt.Println("读取结果：", line)       // 输出：hello,world\n
				  fmt.Println("错误信息：", err)       // 如果正常读到换行符，err 为 nil
				  ```
			- **注意：**
				- 如果在遇到分隔符之前发生错误（比如读到文件末尾），它会返回已读取的部分以及错误（常见如 `io.EOF`）。
				- 只有当返回的字符串不以 `delim` 结尾时，`err` 才不为 `nil`。
				- 如果只是简单按行读取文本，使用 `bufio.Scanner` 可能更方便。
- `net/http`
  heading:: true
	- 用来构建 Web 服务器和客户端程序。
	- `type Client struct {...}`
		- HTTP 客户端。
		- `func (c *Client) Get(url string) (resp *Response, err error)`
			- 向指定的 URL 发送一个 GET 请求，并返回响应（或者错误）。
			- 当 `err` 为 `nil` 时，返回的结果存放在 `resp` 中，其中 `resp.Body` 是响应体，调用者在读取完数据后需手动关闭它，避免资源泄露。
	- `func HandleFunc(pattern string, handler func(ResponseWriter, *Request))`
		- 用于注册一个 HTTP 请求路由及其处理函数到路由器里。每当有请求匹配到 URL，就调用对应的 `handler` 函数进行处理。
	- `func ListenAndServe(addr string, handler Handler) error`
		- 用于启动 HTTP 服务器并监听指定地址上的请求。
		- 它接收一个监听地址和一个路由器（handler），将所有请求交给这个路由器处理。
		- 如果你没有指定路由器（`handler` 传 `nil`），它会使用默认的 `http.DefaultServeMux`。
		- 如果你想使用自己的路由器，只需把实现了 `ServeHTTP` 方法的对象传给 `handler` 参数即可。
		- `ListenAndServe` 会阻塞，直到程序退出或出错。
- `io/ioutil`
  heading:: true
	- 用于简化 IO 操作的工具函数包。
	- `func ReadAll(r io.Reader) ([]byte, error)`
		- 一次性读取一个流中的所有数据，并将其存储到字节切片中。
		- 不适合非常大的数据（会把数据全部读到内存，可能引发 OOM）
- `time`
  heading:: true
	- 用来处理时间和日期相关的操作。
	- `func Now() Time`
		- 获取当前的时间。
	- `func (t Time) Sub(u Time) Duration`
		- 用于计算两个时间点之间的时间间隔。
	- `func Since(t Time) Duration`
		- 用来计算从某个时间点 `t` 到当前时间 `time.Now()` 的时间间隔，等价于 `time.Now().Sub(t)`。
	- `func (t Time) Year() int`
		- 返回 `t` 表示的时间中的年份部分。
- `log`
  heading:: true
	- **用途**：用于记录程序运行过程中的关键信息，在调试时非常有助于追踪问题。
	- **特点：**
		- 默认情况下，`log` 包会自动添加时间戳，记录日志的时间。
	- `func Fatal(v ...any)`
		- 先打印参数内容，然后立即终止程序执行。
- `strconv`
  heading:: true
	- `func ParseFloat(s string, bitSize int) (float64, error)`
		- **作用**：
			- 将字符串 `s` 解析为浮点数。
		- **参数**：
			- `s`：要解析的字符串，比如 `"3.14"`、`"-1.5e2"`。
			- `bitSize`：指定返回结果的精度（位数），可以是 `32` 或 `64`：
				- `bitSize = 32`：结果会被转换为 `float32` 后再提升为 `float64` 返回
				- `bitSize = 64`：直接返回 `float64`
		- **返回值**：
			- 解析得到的 `float64` 数值，以及一个可能的错误。若字符串无效，将返回错误。
	- `func Atoi(s string) (int, error)`
		- **作用：**
			- 将字符串 `s` 解析为一个 `int` 类型的整数。
		- **参数：**
			- `s`：一个表示整数的字符串，比如 `"123"`、`"-42"`。字符串中不能包含空格、字母、小数点或其他非整数字符。
		- **返回值：**
			- 返回解析得到的 `int` 整数值，如果字符串格式合法；
			- 如果字符串格式非法（如包含非数字字符），返回 `0` 和一个非 `nil` 的错误。
- `math/rand`
  heading:: true
	- `func Intn(n int) int`
		- **作用**：
			- 返回一个介于 `0`（包含）到 `n`（不包含）之间的伪随机整数，用于非安全场景下的随机数生成。
		- **参数**：
			- `n`：一个正整数，表示随机数的上限（不包含该值）。即返回值的范围是 `[0, n)`。
		- **返回值**：
			- 一个 `int` 类型的伪随机整数，满足 `0 <= 返回值 < n`。
- `math`
  heading:: true
	- `func Inf(sign int) float64`
		- 用于生成正无穷大或负无穷大的 `float64` 值。
		- 参数 `sign` 用于指定正负号：
			- 如果 `sign >= 0`，返回正无穷大（`+Inf`）；
			- 如果 `sign < 0`，返回负无穷大（`-Inf`）。
- `errors`
  heading:: true
	- `func New(text string) error`
		- **作用**：
			- 创建一个包含指定“错误信息”的基本错误对象。
		- **参数**：
			- `text`：一个字符串类型，表示你想描述的错误信息。
		- **返回值**：
			- 返回一个实现了 `error` 接口的对象，其内部携带了你提供的 `text` 字符串。
			- `fmt.Println` 会自动调用 `err.Error()` 来拿到字符串！
	- `func Is(err, target error) bool`
		- **作用：**
			- `errors.Is` 用于检查某个错误或错误链中是否包含指定的错误。
			- Go 的 `fmt.Errorf` 函数允许我们通过 `%w` 来包装一个错误，形成错误链。`Is` 会沿着错误链进行深度优先遍历，检查错误链中的每一个错误，直到找到目标错误或者遍历完所有错误。
		- **参数：**
			- `err`：被检查的错误。它可能是一个单一的错误，也可能是一个错误链中的一个错误。
			- `target`：目标错误，用于和 `err` 进行比较。
		- **返回值：**
			- `Is` 函数返回一个布尔值：
				- 如果 `err` 或其错误链中的某个错误与 `target` 匹配，则返回 `true`。
				- 如果没有匹配，则返回 `false`。
		-
-