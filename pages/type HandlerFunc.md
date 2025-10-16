- **类型定义：**`type HandlerFunc func(ResponseWriter, *Request)`
- **作用：**
	- `http.HandlerFunc` 是 Go 语言标准库中一个经典的适配器模式应用。它将一个符合 `func(w, r)` 签名的函数，通过类型转换 `http.HandlerFunc(myFunc)`，适配成一个满足 `http.Handler` 接口的对象。
	- Go 语言允许为任何具名类型（包括函数类型）定义方法。通过为 `HandlerFunc` 类型定义 `ServeHTTP` 方法，从而使其满足了 `Handler` 接口的要求：
		- ```go
		  func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
		  	f(w, r)
		  }
		  ```
	- 这段代码的精妙之处在于，方法体中的 `f(w, r)` 直接调用了作为接收者的那个函数本身。因此，任何符合 `func(ResponseWriter, *Request)` 签名的函数，都能通过简单的 `http.HandlerFunc(myFunc)` 类型转换，轻松地适配成一个合法的 `http.Handler`。
	- 在没有 `HandlerFunc` 之前，你必须为每个请求处理逻辑都单独定义一个结构体，并为其实现 `ServeHTTP` 方法，这导致代码冗长且繁琐：
		- ```go
		  // 必须为每个处理器都定义一个结构体
		  type WelcomeHandler struct{}
		  
		  func (h WelcomeHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
		      fmt.Fprintln(w, "Welcome!")
		  }
		  
		  // 实例化结构体并注册
		  http.Handle("/", &WelcomeHandler{})
		  ```
	- 而有了 `HandlerFunc`，你只需要编写一个普通的函数，然后通过类型转换，即可将其注册到路由器中，代码变得非常简洁优雅：
		- ```go
		  // 只需要编写一个普通函数
		  func welcomeHandler(w http.ResponseWriter, r *http.Request) {
		      fmt.Fprintln(w, "Welcome!")
		  }
		  
		  // 通过类型转换，将其适配成一个 Handler 并注册
		  http.Handle("/", http.HandlerFunc(welcomeHandler))
		  ```
	- 这种设计模式不仅是 Go 语言灵活性的体现，也极大地提升了开发效率和代码的可读性，是 Go Web 开发中不可或缺的重要模式。
-