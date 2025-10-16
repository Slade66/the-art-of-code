- **作用：**
	- `HandleFunc` 是 [[func (*ServeMux) Handle]] 方法的一个便利版本。它简化了代码，让你能够直接使用函数来处理 HTTP 请求，而无需为简单的处理逻辑额外创建实现了 `http.Handler` 接口的结构体。
- **方法签名：**
	- ```go
	  func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
	  ```
- **参数：**
	- **`pattern string`：**要匹配的 URL 模式。
	- **`handler func(ResponseWriter, *Request)`：**HTTP 请求处理函数。
- **返回值：**无。它的作用是修改 `http.ServeMux` 实例内部的路由映射关系。
- **注意：**
	- **路由冲突：**和 `Handle` 方法一样，`HandleFunc` 也会在路由模式冲突时 `panic`。
	- **函数签名：**传入 `HandleFunc` 的函数必须严格遵循 `func(http.ResponseWriter, *http.Request)` 的签名。