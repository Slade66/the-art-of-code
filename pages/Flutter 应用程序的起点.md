- 一切从 `main()` 和 `runApp()` 开始：
	- ```dart
	  void main() {
	    runApp(MyApp());
	  }
	  ```
- `main()` 是 Dart 程序的入口，就像 C/C++ 或 Java 中的 `main()`。
- `runApp()` 是 Flutter 提供的一个函数，它的作用是：启动整个应用，并将你传入的 Widget 显示到屏幕上。
- **`runApp()` 必须传入一个 App 级别的 Widget：**
	- 直接将普通 Widget（如 `Text`）传给 `runApp()` 会导致程序错误，因为它缺乏构建完整应用所需的基础架构。这就像盖房子时，只在地基上放了一块砖头，却没有搭建钢筋、墙体和屋顶。
	- 一个完整的 App 通常需要以下“基础设施”：
		- **主题（Theme）：**统一的颜色、字体、按钮样式等。若缺失，组件会使用简陋的默认样式。
		- **文字方向（Directionality）：**决定文本是从左到右（LTR）还是从右到左（RTL）渲染。
		- **路由（Routing）：**用于页面间的跳转与管理。
		- **基本页面结构：**如顶部的应用栏（AppBar）和页面主体（Body）等。
		- **国际化支持：**便于应用根据用户语言环境自动切换显示语言。
	- 这些“基础设施”通常由专门的 App 级别 Widget 提供。在实际开发中，我们一般将以下 Widget 作为参数传给 `runApp()`：
		- **`MaterialApp`**：实现 Google 的 Material Design 风格，是最常用的选择。
		- **`CupertinoApp`**：实现苹果的 iOS（Cupertino）设计风格，适用于构建 iOS 风格的应用。
-