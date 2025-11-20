## 测试文件的规范
	- Go 语言的测试文件必须遵循严格的命名规范，以便 Go 工具链能够识别它们。
	- **测试文件：**文件名必须以 `_test.go` 结尾。
	- **测试函数：**函数名必须以 `Test` 开头，并接受一个参数：`*testing.T`。
	- **包名：**测试文件通常与被测试的源代码文件位于同一个包内。
- ## *testing.T
	- `*testing.T` 提供了控制测试流程和报告结果的方法。
	- `t.Errorf(...)`：记录失败信息，测试继续执行。
	- `t.Fatalf(...)`：记录失败信息，并立即停止当前测试函数的执行。
	- `t.Run(name, fn)`：创建一个子测试。
- ## 运行测试
	- `go test`：运行当前目录下所有 `_test.go` 文件中的测试函数。
	- `go test ./...`：运行所有子目录中的测试。
	- `go test -v`：运行测试并输出详细信息，显示每个测试函数的运行情况。
	- `go test -run TestAddTableDriven`：仅运行名称匹配指定模式的测试函数。
- ## 代码示例
	- `calc.go`：
		- ```go
		  package calculator
		  
		  // Add 接收两个整数并返回它们的和
		  func Add(a, b int) int {
		  	return a + b
		  }
		  ```
	- `calc_test.go`：
		- ```go
		  package calculator
		  
		  import (
		  	"testing" // 导入标准的 testing 包
		  )
		  
		  // TestAddFunction 是对 Add 函数的测试
		  // 函数签名必须是 func TestXxx(t *testing.T)
		  func TestAddFunction(t *testing.T) {
		  	// 1. 定义期望的输入和输出
		  	a := 5
		  	b := 7
		  	expected := 12
		  
		  	// 2. 调用被测试的函数
		  	actual := Add(a, b)
		  
		  	// 3. 断言 (Assertion)：检查实际结果是否符合预期
		  	if actual != expected {
		  		// 如果结果不匹配，使用 t.Errorf 记录失败信息
		  		// t.Errorf 会标记测试失败，但会继续执行后续代码
		  		t.Errorf("Add(%d, %d) 期望得到 %d, 但实际得到 %d", a, b, expected, actual)
		  	}
		  
		  	// 另一种测试用例（成功的情况）
		  	if Add(0, 0) != 0 {
		  		t.Errorf("Add(0, 0) 失败")
		  	}
		  }
		  ```
	- 使用 `t.Run` 进行表格驱动测试：
		- 在 Go 中，我们通常使用表格驱动测试（Table Driven Tests）来简洁地处理多个测试用例。
		- ```go
		  package calculator
		  
		  import (
		  	"testing"
		  )
		  
		  func TestAddTableDriven(t *testing.T) {
		  	// 1. 定义测试用例表格
		  	tests := []struct {
		  		name     string // 测试用例的名称
		  		a        int    // 输入 a
		  		b        int    // 输入 b
		  		expected int    // 期望的输出
		  	}{
		  		{"PositiveNumbers", 1, 2, 3},
		  		{"ZeroAndNegative", 10, -5, 5},
		  		{"TwoNegatives", -5, -3, -8},
		  		{"ZeroInput", 0, 0, 0},
		  	}
		  
		  	// 2. 遍历表格，为每个用例执行 t.Run
		  	for _, tt := range tests {
		  		// t.Run 允许您创建子测试。当某个子测试失败时，只会报告该子测试失败。
		  		t.Run(tt.name, func(t *testing.T) {
		  			actual := Add(tt.a, tt.b)
		  
		  			if actual != tt.expected {
		  				t.Errorf("输入 (%d, %d) 期望得到 %d, 实际得到 %d", tt.a, tt.b, tt.expected, actual)
		  			}
		  		})
		  	}
		  }
		  ```
-