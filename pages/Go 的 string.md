- **基本概念：**
	- 字符串用于表示由一系列字符组成的文本数据。
	- 字符串是只读的字节切片（`[]byte`）。
	- 在 Go 中，字符串是不可变的，意味着一旦创建，其内容无法修改。
	- 字符串的默认值为 `""`（空字符串）。
	- Go 中的字符串可以通过 `+` 运算符连接：
		- ```go
		  fmt.Println("go" + "lang") // 输出：golang
		  ```
- **使用 `range` 迭代字符串：**
	- `range` 可以用于迭代字符串中的每个字符。
	- ```go
	  for idx, runeValue := range s {
	      fmt.Printf("%#U starts at %d\n", runeValue, idx)
	  }
	  ```
	- `range s` 会迭代字符串 `s` 中的每个字符，其中 `runeValue` 是字符的 Unicode 代码点（`rune` 类型），`idx` 是字符在字符串中的起始字节索引。
-