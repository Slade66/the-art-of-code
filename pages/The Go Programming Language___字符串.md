- **字符串是什么？**
  collapsed:: true
	- 在 Go 中，字符串用于表示由字符序列组成的一串文本数据。
	- Go 语言的字符串类型是 `string`。
- **字符串的底层原理：**
  collapsed:: true
	- **不可变性：**
		- Go 语言的字符串是UTF-8 编码的不可变的字节序列，一旦创建了字符串，其内容无法修改。
			- **注意：**但只是字符串不变而已，持有字符串的变量可以指向一个新的字符串值。
		- 任何看起来在修改字符串的操作，例如拼接或替换，实际上都是在创建一个全新的字符串。
	- **底层结构：一个只读的字节切片**
		- Go 语言的字符串在内部并非一个复杂的对象，而是一个极为简洁的两个字段长的数据结构：
			- ```go
			  // Go 运行时中字符串的内部表示
			  type _string struct {
			      elements *byte // 指向底层字节数组的指针
			      len      int   // 字节数量
			  }
			  ```
		- 将字符串作为参数传递给函数时，成本非常低廉，因为它只涉及复制这个两字长的数据头，而非整个底层的字节数组。
	- `len(s)` 返回的是字符串字节的数量，而不是字符的数量。
		- **注意：**要想返回字符数，先转换成 `[]rune`。
		- ```go
		  s := "Hello嘿"
		  fmt.Printf("字符串长度: %d\n", len(s)) // 输出 8，因为前五个英文字符占 5 个字节，中文字符占 3 个字节
		  ```
	- 可以像数组一样通过索引访问字符串中的字节，返回的是 `byte` 类型。
		- ```go
		  s := "Hello"
		  fmt.Printf("s[0] 的值: %v, 类型: %T\n", s[0], s[0]) // 输出: s[0] 的值: 72, 类型: uint8 (即 byte)
		  ```
- **字符串的字面量：**
  collapsed:: true
	- 字符串的默认值为 `""`（空字符串）。
	- **双引号 `"`：** 用于普通的字符串，支持转义字符（如 `\n`, `\t`, `\\`）。
	- **反引号 \``（原始字符串字面量）：** 用于创建跨多行、包含换行符的字符串，它不会对转义字符进行转义。
- **字符串的拼接：**
  collapsed:: true
	- **`+` 操作符**：对于连接少量、固定数量的字符串，这种方式简单易读。但由于字符串的不可变性，`s1 + s2 + s3` 会导致生成中间字符串，因此在循环中使用效率低下 。
	- **`strings.Builder`**：从多个片段构建字符串时性能最高的方法，尤其是在循环中。它内部使用一个可变的 `byte` 缓冲区，该缓冲区可以高效地增长，从而最大限度地减少内存分配和数据复制。最后通过 `String()` 方法一次性生成最终的不可变字符串。
	- **`fmt.Sprintf`**：功能强大，适用于复杂的格式化场景，但由于需要解析格式化字符串和进行反射操作，对于简单的拼接而言通常是效率最低的选择。
	- **`strings.Join`**：当你需要用一个分隔符将一个字符串切片连接起来时，这是一个非常高效的选择。
- **使用 `range` 迭代字符串中的每个字符：**
  collapsed:: true
	- `for...range` 循环对字符串有特殊处理。它在每次迭代中解码一个 UTF-8 符文，返回该 `rune` 的值以及该字符的第一个字节在字符串中的索引。对于多字节字符串，这个索引值将不是连续的。
	- ```go
	  s := "你好 Go"
	  
	  fmt.Println("--- 按字节遍历 ---")
	  for i := 0; i < len(s); i++ {
	      fmt.Printf("字节索引 %d: %v\n", i, s[i]) // 可能会将一个中文字符的字节拆开
	  }
	  
	  fmt.Println("--- 按字符 (Rune) 遍历 ---")
	  for index, char := range s {
	      fmt.Printf("字符索引 %d: %c (Unicode: %U)\n", index, char, char)
	  }
	  
	  --- 按字节遍历 ---
	  字节索引 0: 228
	  字节索引 1: 189
	  字节索引 2: 160
	  字节索引 3: 229
	  字节索引 4: 165
	  字节索引 5: 189
	  字节索引 6: 32
	  字节索引 7: 71
	  字节索引 8: 111
	  --- 按字符 (Rune) 遍历 ---
	  字符索引 0: 你 (Unicode: U+4F60)
	  字符索引 3: 好 (Unicode: U+597D)
	  字符索引 6:   (Unicode: U+0020)
	  字符索引 7: G (Unicode: U+0047)
	  字符索引 8: o (Unicode: U+006F)
	  ```
	-
- **获取 Go 语言字符串中的某个“字符”：**
  collapsed:: true
	- Go 字符串是 UTF-8 编码的字节序列，一个字符（尤其是中文、表情符号等）可能占用多个字节。
	- **获取第 N 个 字节：**
		- 如果你只需要获取字符串中底层字节序列的第 N 个字节，可以使用数组索引的方式。但对于非 ASCII 字符可能会得到乱码。
		- **语法：** `s[i]`，返回 `byte` 类型。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      sAscii := "Hello"
		      
		      // 获取第 0 个字节 (H 的 ASCII 码是 72)
		      b0 := sAscii[0]
		      fmt.Printf("sAscii[0]: %v (类型: %T)\n", b0, b0) 
		      fmt.Printf("sAscii[0] 对应的字符: %c\n", b0) 
		      // 输出: 72 (类型: uint8)
		      // 输出: H
		  
		      sUtf8 := "你好Go" 
		      // "你" 占 3 个字节：E4 BD A0
		      
		      // 尝试直接获取第 0 个字节
		      b1 := sUtf8[0]
		      fmt.Printf("sUtf8[0]: %v (类型: %T)\n", b1, b1) 
		      fmt.Printf("sUtf8[0] 对应的字符: %c\n", b1) 
		      // 输出: 228 (类型: uint8)
		      // 输出: ä (这是 UTF-8 编码的第一个字节被单独解析的结果，通常是乱码)
		  }
		  ```
	- **获取第 N 个 Unicode 字符 (Rune)：**
		- 如果你想正确地获取用户可见的第 $N$ 个字符（`rune`），你需要将字符串转换为 `[]rune` 切片，然后进行索引。
		- **语法：**`[]rune(s)[i]`，返回 `rune` 类型。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      sUtf8 := "你好Go!" 
		      
		      // 1. 将字符串转换为 rune 切片
		      runes := []rune(sUtf8)
		      
		      // 2. 使用索引获取第 N 个字符
		      
		      // 获取第 0 个字符 ('你')
		      r0 := runes[0]
		      fmt.Printf("runes[0]: %c (类型: %T)\n", r0, r0)
		      // 输出: 你 (类型: int32)
		  
		      // 获取第 1 个字符 ('好')
		      r1 := runes[1]
		      fmt.Printf("runes[1]: %c\n", r1)
		      // 输出: 好
		      
		      // 获取最后一个字符 ('!')
		      lastRune := runes[len(runes) - 1]
		      fmt.Printf("最后一个字符: %c\n", lastRune)
		      // 输出: !
		      
		      // 如果需要将单个 rune 转换回 string
		      sChar := string(r1)
		      fmt.Printf("转换回字符串: %s (长度: %d)\n", sChar, len(sChar))
		      // 输出: 好 (长度: 3，因为“好”仍是 3 个字节)
		  }
		  ```
- **把字符序列或字节序列转换为字符串：**
  collapsed:: true
	- **`string([]byte)`：**将字节序列解释为 UTF-8 字符串。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      // 1. 字节数组 ([]byte)
		      byteArray := []byte{'H', 'e', 'l', 'l', 'o', ',', ' ', 'G', 'o'}
		      
		      // 直接类型转换
		      s1 := string(byteArray) 
		      
		      fmt.Println("字节数组转字符串:", s1) // 输出: Hello, Go
		      
		      // 2. 字节切片 (更常用)
		      byteSlice := []byte{0xE4, 0xBD, 0xA0, 0xE5, 0xA5, 0xBD} // 这是 "你好" 的 UTF-8 编码
		      
		      s2 := string(byteSlice)
		      
		      fmt.Println("UTF-8 字节转字符串:", s2) // 输出: 你好
		  }
		  ```
		- **关键点：为什么可以直接转换？**
			- Go 语言的字符串在底层就是**只读的字节序列**。因此，这种类型转换非常高效，它不会发生数据拷贝，编译器只是创建了一个新的 `string` 结构体，引用了字节切片所指向的内存区域。
	- **`string([]rune)`：**将 Unicode 码点（字符）序列转换为 UTF-8 字符串。
		- ```go
		  package main
		  
		  import "fmt"
		  
		  func main() {
		      // 字符切片 ([]rune)
		      runeSlice := []rune{'你', '好', ' ', 'G', 'o', '!'}
		      
		      // 直接类型转换
		      s := string(runeSlice)
		      
		      fmt.Println("字符切片转字符串:", s) // 输出: 你好 Go!
		      
		      // 验证类型
		      fmt.Printf("s 的类型: %T\n", s) // 输出: string
		  }
		  ```
- **字符串切片的内存保留陷阱：**
  collapsed:: true
	- 对字符串进行切片操作（`substr := s[i:j]`）的效率极高，因为它**不会复制底层数据**。新的子字符串 `substr` 只是一个指向与原始字符串 `s` 相同字节数组的 `StringHeader`，但具有不同的 `len` 和指针偏移值。
	- 然而，这也带来了一个常见的“内存保留陷阱”。假设你有一个非常大的字符串（例如，一个 1GB 的文件读入内存），然后你只取了其中一小部分（`smallStr := bigStr[0:10]`）。尽管你只对这一小段感兴趣，但垃圾回收器（GC）无法释放那 1GB 的数组，因为 `smallStr` 仍然持有对它的引用。
	- Go 1.18 之后，`strings.Clone` 函数为此提供了解决方案。`clonedStr := strings.Clone(smallStr)` 会创建一个新的、大小恰到好处的字节数组，并将 `smallStr` 的内容复制过去。这样，原始的大数组就可以被垃圾回收器回收了。
- **修改字符串中的某个字节会导致编译错误**
  collapsed:: true
	- ```go
	  var s string = "hello"
	  s[0] = 'H' // .\main.go:14:2: cannot assign to s[0] (neither addressable nor a map index expression)
	  ```
	- 字符串的下标访问（`s[0]`）返回的是字节值的副本，而非可修改的内存地址，因此无法赋值。
- **不能对字符串的字节元素取地址**
  collapsed:: true
	- ```go
	  s := "Hello"
	  println(&s[0]) // .\main.go:5:11: invalid operation: cannot take address of s[0] (value of type byte)
	  ```
	- Go 语言禁止获取字符串内容的地址，因为字符串内容是不可变的，允许取地址可能会绕过这一限制。
- utf8.RuneCountInString() 计算字符串中的字符个数
  collapsed:: true
	- `utf8.RuneCountInString()` 用来计算字符串中包含多少个 Unicode 代码点（Rune），并不是数你眼里看到的“字”的数量。
	- 在泰语（以及很多其他语言）中，看似一个完整的字形，实际上可能由多个 Unicode 字符组合而成。
	- 比如 สวัสดี 在计算机底层实际上是由 6 个 独立的 Unicode 代码点组成的：
		- ```
		  +----+------+-----------+---------------+--------------------+
		  |序号|字形  |Unicode    |描述           |类型                |
		  +----+------+-----------+---------------+--------------------+
		  | 1  | ส    |0xE0E2A    |SO SUEA        |辅音                |
		  | 2  | ว    |0xE0E27    |WO WAEN        |辅音                |
		  | 3  | ั    |0xE0E31    |MAI HAN-AKAT   |元音符号(组合)      |
		  | 4  | ส    |0xE0E2A    |SO SUEA        |辅音                |
		  | 5  | ด    |0xE0E14    |DO DEK         |辅音                |
		  | 6  | ี    |0xE0E35    |SARA II        |元音符号(组合)      |
		  +----+------+-----------+---------------+--------------------+
		  ```
	- 人类阅读时，会习惯把辅音和附着在它上面/下面的元音符号看作一个整体，但对于 Go 语言的 `utf8.RuneCountInString` 来说，它不关心显示效果，它只数有多少个独立的 Unicode 编码。因为有两个字符是“修饰符”（元音），它们也是独立的编码，所以总数是 6。
-