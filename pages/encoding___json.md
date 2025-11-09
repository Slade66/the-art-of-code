- **有什么用？**
	- `encoding/json` 包实现了 JSON 标准（RFC 7159）的编解码。
		- **编码：**将 Go 结构体转换为标准的 JSON 文本。
		- **解码：**将 JSON 数据解析后填充到指定的 Go 结构体变量中。
- **注意：**
	- **重复键：**若 JSON 中存在重复键，后面的值会覆盖或合并前面的值；其中 `map` 和 `struct` 类型会合并，其他类型则被替换。
	- **字段匹配：**字段名匹配时不区分大小写，例如 JSON 的 `"Name"` 能匹配 Go 结构体中的 `name`。
	- **多余字段：**JSON 中出现结构体未定义的字段时，这些字段会被自动忽略。
	- **大数字的麻烦：**如果 JSON 里有一个非常非常大的整数，你把它读成小数（`float64`），可能会丢失精度。
- `func Unmarshal(data []byte, v any) error`
  collapsed:: true
	- **作用：**将接收到的、格式化的 JSON 文本数据，自动解析并填充到你指定的 Go 语言变量（通常是结构体、切片或 Map）中，使其成为 Go 语言中可以方便操作的数据结构。
	- **参数：**
		- `data []byte`：必须是包含有效的 JSON 编码数据的字节切片。
		- `v any`：必须是一个非空的指针，指向你希望填充数据的 Go 变量（结构体、切片、Map 等）。
	- **注意：**
		- **无法给小写开头的结构体字段赋值：**在 Go 中，只有以大写字母开头的结构体字段才是可导出的，`encoding/json` 等外部包才能访问和修改它们。若字段名以小写字母开头（不可导出），即使 JSON 数据完全正确，`encoding/json` 也无法为这些字段赋值，解析结果中它们会保持默认零值（如字符串为 `""`、数字为 `0`），从而给人一种“数据没有解析到”的错觉。
	- **代码示例：**
	  collapsed:: true
		- ```go
		  package main
		  
		  import (
		  	"encoding/json"
		  	"fmt"
		  	"log"
		  )
		  
		  // 1. 定义一个 Go 结构体来接收数据
		  type User struct {
		  	ID   int    `json:"id"`
		  	Name string `json:"name"`
		  }
		  
		  func main() {
		  	// 2. 准备 JSON 数据（必须是 []byte 类型）
		  	jsonData := []byte(`{"id": 101, "name": "Alice"}`)
		  
		  	// 3. 创建一个变量（指针形式）来接收解析结果
		  	var u User 
		  
		  	// 4. 调用 Unmarshal 进行解析
		  	err := json.Unmarshal(jsonData, &u) 
		  
		  	if err != nil {
		  		log.Fatal(err) // 如果解析失败，直接打印错误并退出
		  	}
		  
		  	// 5. 打印结果
		  	fmt.Printf("解析成功:\n")
		  	fmt.Printf("ID: %d\n", u.ID)
		  	fmt.Printf("Name: %s\n", u.Name)
		  }
		  ```
- `func Marshal(v any) ([]byte, error)`
  collapsed:: true
	- **作用：**将 Go 语言中的值（变量、结构体、切片、Map 等）转换为其对应的 JSON 编码格式的字节切片。
	- **参数：**
		- `v any`：任何 Go 语言的值（变量或结构体实例）。
	- **返回值：**
		- `[]byte`：转换后的 JSON 格式的字节切片。
	- **注意：**
		- **只编码可导出字段：**`Marshal` 只会编码结构体中以大写字母开头的字段（可导出字段）。
		- `encoding/json` 会把 nil 切片编码成 `null`，而空切片会编码成 `[]`。
		- **JSON Tag：**字段标签（`json:"..."`）用于控制 JSON 中的键名和行为：
			- `json:"keyName"`：指定该字段在 JSON 中对应的键名。
			- `json:"-"`：忽略该字段，不参与 JSON 编码或解码。
			- `json:",omitempty"`：当字段值为空（如 `false`、`0`、`nil` 或零长度的切片、Map、字符串）时，省略该字段。
			- `json:",omitzero"`：当字段值为零值，或字段类型实现了 `IsZero()` 方法且返回 `true` 时，省略该字段。
- `func MarshalIndent(v any, prefix, indent string) ([]byte, error)`
  collapsed:: true
	- **作用：**
		- `MarshalIndent` 与 `Marshal` 类似，但会为 JSON 输出添加缩进和换行。
		- `prefix` 会出现在每一行的最前面；
		- `indent` 会根据 JSON 的层级重复，用于表示缩进；
		- 这样可以让 JSON 输出更加易读，尤其适合打印调试或日志输出。
-