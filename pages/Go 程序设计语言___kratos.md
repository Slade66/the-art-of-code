-
- `func (s *Server) WalkRoute(fn WalkRouteFunc) error`
	- **源码：**
		- ```go
		  // WalkRouteFunc is the type of the function called for each route visited by Walk.
		  type WalkRouteFunc func(RouteInfo) error
		  
		  // RouteInfo is an HTTP route info.
		  type RouteInfo struct {
		  	Path   string
		  	Method string
		  }
		  ```
	- **作用：**遍历当前 HTTP 服务注册的所有路由（包括 Proto 生成的 + 手动注册的），把每一个真实接口的 Method + Path 传给你传入的回调函数。
	- **注意：**
		- 只要你的回调返回 error，整个遍历就立刻停止。
		- 使用 `func (s *Server) Handle(path string, h http.Handler)` 注册的路由不会被遍历。
			- 因为这种写法没有绑定任何 HTTP Method，而 `WalkRoute` 内部会调用 `route.GetMethods()`。如果路由没有绑定方法，`GetMethods()` 会报错，该路由就会被跳过。
			- 需要改用 Router API：
				- ```go
				  r := srv.Route("/")
				  
				  r.GET("/healthz", func (c http.Context) error {
				    return c.JSON(200, `{"message":"ok"}`)
				  })
				  ```
	- **示例：**
		- ```go
		  package main
		  
		  import (
		  	"fmt"
		  
		  	"github.com/go-kratos/kratos/v2/transport/http"
		  )
		  
		  func main() {
		  	srv := http.NewServer()
		  
		  	r := srv.Route("/")
		  
		  	r.GET("/healthz", func (c http.Context) error {
		  		return c.JSON(200, `{"message":"ok"}`)
		  	})
		  
		  	r.POST("/user", func (c http.Context) error {
		  		return c.JSON(200, `{"username":"lyz"}`)
		  	})
		  
		  	srv.WalkRoute(func (rf http.RouteInfo) error {
		  		fmt.Printf("%s, %s\n", rf.Method, rf.Path)
		  		return nil
		  	})
		  }
		  ```
-