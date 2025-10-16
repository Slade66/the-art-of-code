-
- `strconv` 包是 "string conversion" 的缩写，专门用于在字符串与基本数据类型之间进行相互转换。
- #### `func Itoa(i int) string`
	- `Itoa` 是 "Integer to ASCII" 的缩写，用于将一个 `int` 类型的整数转换为它对应的十进制字符串表示。
	- **示例代码：**
		- ```go
		  num := 12345
		  str := strconv.Itoa(num)
		  
		  fmt.Printf("数字: %d, 类型: %T\n", num, num)
		  fmt.Printf("字符串: \"%s\", 类型: %T\n", str, str)
		  ```
- #### `func Atoi(s string) (int, error)`
	- `Atoi` 是 "ASCII to Integer" 的缩写，用于将一个表示十进制数字的字符串转换为 `int` 类型的整数。
	- **示例代码：**
		- ```go
		  str := "12345"
		  num, err := strconv.Atoi(str)
		  if err != nil {
		    fmt.Println("转换出错:", err)
		    return
		  }
		  
		  fmt.Printf("字符串: \"%s\", 类型: %T\n", str, str)
		  fmt.Printf("数字: %d, 类型: %T\n", num, num)
		  ```
-