- `go test` 会在包中查找所有以 `_test.go` 结尾的文件，并执行其中所有以 `Test` 开头、参数为 `*testing.T` 的测试函数。
- 在发布新版本给用户前，你可能已经测试了所有新功能，以确保它们正常运行。但你是否也检查了旧功能，确保它们没有在更新中被意外破坏呢？
- 自动化测试程序是一个独立的程序，用于执行主程序中的各个组件，并验证它们的行为是否符合预期。
- 自动化测试通过一组特定的输入运行代码，并检查其输出是否与预期结果匹配。只要输出与期望一致，测试就会“通过”。
- `go build` 和 `go install` 这两个 Go 工具在编译程序时，会自动忽略所有以 `_test.go` 结尾的文件。这意味着，即使测试代码与主程序代码位于同一包目录下，也不会被编译进最终的可执行文件中。
- 测试代码不必与被测试的代码位于同一个包中；但如果你需要访问包中未导出的类型或函数，则必须将测试代码置于同一包内。
- **代码示例：**
	- ```go
	  package main
	  
	  import "testing"
	  
	  // TestAdd 测试 Add 函数
	  func TestAdd(t *testing.T) {
	  	sum := Add(1, 2)
	  	expected := 3
	  
	  	if sum != expected {
	  		t.Errorf("Add(1, 2) 期望得到 %d, 实际得到 %d", expected, sum)
	  	}
	  }
	  
	  // TestAddTableDriven 表格驱动测试示例
	  // 表格驱动测试适用于需要针对同一函数验证多组输入输出组合的情况。
	  func TestAddTableDriven(t *testing.T) {
	  	var tests = []struct {
	  		a        int
	  		b        int
	  		expected int
	  	}{
	  		{1, 2, 3},
	  		{0, 0, 0},
	  		{-1, 1, 0},
	  		{100, 200, 300},
	  	}
	  
	  	for _, test := range tests {
	  		if output := Add(test.a, test.b); output != test.expected {
	  			t.Errorf("Add(%d, %d) 期望得到 %d, 实际得到 %d", test.a, test.b, test.expected, output)
	  		}
	  	}
	  }
	  ```
-