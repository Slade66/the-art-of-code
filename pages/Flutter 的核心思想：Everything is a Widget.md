- 在 Flutter 中，你能看到的一切几乎都是 Widget。这不仅仅包括按钮（Button）、文本（Text）、输入框（Input Field）这些可见的 UI 元素，还包括一些“不可见”的东西，比如：
	- **布局（Layout）：**`Row`（行）、`Column`（列）、`Padding`（内边距）、`Center`（居中）都是 Widget。它们的作用是告诉它们的子 Widget 如何排列、对齐和显示。
	- **结构（Structure）：**`MaterialApp`（App 的根）、`Scaffold`（页面的基本骨架）是 Widget。
	- **状态管理（State Management）：**`InheritedWidget` 用来在 Widget 树中传递数据。
	- **手势检测（Gesture Detection）：**`GestureDetector` 是一个不可见的 Widget，但它可以包裹任何其它 Widget，使其能够响应点击、拖动等手势。
-