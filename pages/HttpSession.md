`HttpSession` 的创建时机
heading:: true
	- 在 Servlet 或 JSP 中
	  heading:: true
		- `HttpSession` 并不会在每次请求中自动创建。如果访问的资源未调用 `getSession()` 方法，或使用了 `request.getSession(false)` 且当前不存在 Session，则不会创建新的 Session。
		- 只有在明确需要时，即调用 `request.getSession()` 或 `request.getSession(true)`，并且当前请求未关联有效的 Session 时，Servlet 容器才会创建一个新的 `HttpSession` 对象。
		- 通过 `HttpServletRequest` 的 `getSession()` 方法来获取或创建 Session。该方法有两种形式：
			- `request.getSession()` 或 `request.getSession(true)`：
				- 检查当前请求是否已关联有效的 Session（即浏览器发送了有效的 Session ID 且服务器中有对应的 Session）。
				- 若存在有效的 Session，则返回该对象；
				- 若不存在（如首次访问、Session 已过期或浏览器未携带 Session ID），则容器会：
					- 创建一个新的 `HttpSession` 对象；
					  logseq.order-list-type:: number
					- 生成新的 Session ID；
					  logseq.order-list-type:: number
					- 设置对应的 Cookie 返回给浏览器；
					  logseq.order-list-type:: number
					- 返回这个新创建的 Session。
					  logseq.order-list-type:: number
			- `request.getSession(false)`：
				- 同样检查当前请求是否已关联有效 Session；
				- 若存在有效 Session，则返回该对象；
				- 若不存在，则不会创建新 Session，而是返回 `null`。
				- 这种方式常用于只想获取已存在的 Session，而不希望在不存在时自动创建的场景，例如判断用户是否已登录。
	- 在 Spring MVC 中
	  heading:: true
		- 在 Spring MVC 中，`HttpSession` 的创建与管理机制本质上与底层 Servlet API 一致。Spring MVC 并没有独立的 Session 实现，而是直接依赖 Servlet 容器提供的 `HttpSession`。
		- 与 Servlet 一样，Spring MVC 中的 `HttpSession` 也是在“需要时但尚不存在”的情况下才会被创建。
		- 不同之处在于，Spring MVC 提供了更为便捷和声明式的方式与 `HttpSession` 交互，开发者通常无需直接操作 `HttpServletRequest` 来获取 Session。
		- Controller 方法中注入 `HttpSession` 参数：
			- 它会查看当前进来的 `HttpServletRequest` 是否已经携带了一个有效的 Session ID。
			- 如果请求中包含有效的 Session ID，且服务器端能找到对应的活动 `HttpSession` 对象，则该对象会直接注入到方法中的 `session` 参数中。
			- 如果请求中没有 Session ID，或该 ID 无效、已过期，或者服务器找不到对应的 `HttpSession` 对象，Servlet 容器将会创建一个新的 `HttpSession` 对象，为其生成新的 Session ID，并通过响应头的 `Set-Cookie` 指令告知客户端保存该 ID。随后，这个新创建的 `HttpSession` 对象将被注入到方法的 `session` 参数中。