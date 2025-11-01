- | **函数** | **描述** | **示例** |
  | ---- | ---- | ---- |
  | `Contains` | 检查字符串是否包含子串 | `strings.Contains("abc", "b")` -> `true` |
  | `HasPrefix` | 检查字符串是否以指定前缀开头 | `strings.HasPrefix("abc", "a")` -> `true` |
  | `HasSuffix` | 检查字符串是否以指定后缀结尾 | `strings.HasSuffix("abc", "c")` -> `true` |
  | `Index` | 查找子串第一次出现的位置（字节索引） | `strings.Index("abcba", "b")` -> `1` |
  | `Join` | 使用分隔符连接字符串切片 | `strings.Join([]string{"a", "b"}, "-")` -> `"a-b"` |
  | `Split` | 使用分隔符拆分字符串为切片 | `strings.Split("a-b-c", "-")` -> `["a", "b", "c"]` |
  | `Replace` | 替换字符串中的子串 | `strings.Replace("ooxx", "o", "a", -1)` -> `"aaxx"` |
  | `ToLower`/`ToUpper` | 转换为小写/大写 | `strings.ToLower("GO")` -> `"go"` |
  | `TrimSpace` | 移除字符串首尾的空白字符 | `strings.TrimSpace(" go ")` -> `"go"` |
-