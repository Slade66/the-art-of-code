-
- `func Replace(s, old, new string, n int) string`
	- `strings.Replace` 函数会返回一个新的字符串，这个新字符串是把原始字符串 `s` 中指定数量的子字符串 `old` 替换为 `new` 之后得到的结果。
	- **参数详解：**
		- `s string`：这是原始字符串，也就是你想要进行替换操作的字符串。
		- `old string`：这是被替换的子字符串。函数会在 `s` 中查找所有出现的 `old`。
		- `new string`：这是新的子字符串，用于替换所有找到的 `old`。
		- `n int`：这个参数用来控制替换的次数，它的取值直接决定了函数的行为：
			- `n > 0`：只替换前 n 个匹配到的 `old` 子字符串。例如，如果 `n` 是 `1`，就只替换第一个找到的 `old`。
			- `n = 0`：不进行任何替换，函数会直接返回原始字符串 `s` 的一个副本。
			- `n < 0`：替换所有匹配到的 `old` 子字符串。通常我们会传入 `-1` 来代表“全部替换”。
	- **返回值详解：**
		- `string`：函数会返回一个经过替换操作后的新字符串。
	- **注意：**
		- Go 中的字符串是不可变的，所以这个函数不会修改原始字符串 `s`，而是创建一个新的字符串。
		- 当一个 `old` 子串被替换后，搜索下一个 `old` 的位置会从刚替换完的 `new` 字符串之后开始，而不是从 `old` 的第二个字符开始。
		- 当参数 `old` 是一个空字符串 `""` 时，函数会在字符串的开头和每个字符之后都进行一次匹配和插入。
		- `strings.Replace` 是大小写敏感的。`"Go"` 和 `"go"` 会被视为两个不同的字符串。
	- **示例代码：**
		- ```go
		  s := "hello world"
		  
		  // 1. n > 0，只替换前 n 个
		  fmt.Println(strings.Replace(s, "l", "L", 1)) // heLlo world
		  fmt.Println(strings.Replace(s, "l", "L", 2)) // heLLo world
		  
		  // 2. n = 0，不替换，返回原始字符串副本
		  fmt.Println(strings.Replace(s, "l", "L", 0)) // hello world
		  
		  // 3. n < 0，替换所有
		  fmt.Println(strings.Replace(s, "l", "L", -1)) // heLLo worLd
		  ```
- [[strings.ReplaceAll]]
-