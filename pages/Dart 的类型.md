基本概念
heading:: true
	- Dart 是一种类型安全的语言，这意味着一旦声明了变量的类型，它将始终保持该类型不变。
	- Dart 支持类型推断，在许多情况下可以省略类型声明，由编译器自动推断出正确的类型。
- 内置核心类型
  heading:: true
	- **数值类型**
		- Dart 的数值类型主要包括两种：`int` 和 `double`，它们都是 `num` 的子类。这意味着你可以声明一个 `num` 类型的变量，它既可以存储整数，也可以存储浮点数。
		- **`int`：**用于表示整数。它的大小取决于运行的平台：在原生平台（比如 Flutter 的移动端或桌面端）上是 64 位的；而在 Web 平台上，由于 Dart 会被编译成 JavaScript，所以整数实际上是用 JavaScript 的数字表示的，虽然表面上是浮点数，但可以安全地表示最多 53 位的整数。
		- **`double`：**用于表示小数，是一种 64 位的双精度浮点数，符合 IEEE 754 标准，能表示绝大多数常见的小数值。
		- **代码示例：**
			- ```dart
			  // 使用 var 进行类型推断
			  var integerNumber = 10;      // 推断为 int
			  var doubleNumber = 10.5;     // 推断为 double
			  
			  // 显式声明类型
			  int age = 20;
			  double price = 99.99;
			  
			  // num 类型可以持有 int 或 double
			  num someNumber = 100;       // 初始为 int
			  print(someNumber); // 输出: 100
			  
			  someNumber = 3.14;        // 现在赋值为 double
			  print(someNumber); // 输出: 3.14
			  
			  // 字符串和数字之间的转换
			  // String -> int
			  var one = int.parse('1');
			  assert(one == 1); // assert 用于断言，如果条件为 false 会抛出异常
			  
			  // String -> double
			  var onePointOne = double.parse('1.1');
			  assert(onePointOne == 1.1);
			  
			  // int -> String
			  String oneAsString = 1.toString();
			  assert(oneAsString == '1');
			  
			  // double -> String，保留两位小数
			  String piAsString = 3.14159.toStringAsFixed(2);
			  assert(piAsString == '3.14');
			  ```
	- **字符串**
		- **多行字符串：**使用三个单引号或三个双引号。
		- **字符串插值：**可以在字符串中用 `${表达式}` 来插入一个变量或表达式。如果只是一个变量名，也可以省略大括号，直接写成 `$变量名`。
		- **原始字符串：**在字符串前加上 `r`，可以使字符串中的特殊字符（如 `\n`, `\t`）失效。
		- **代码示例：**
			- ```dart
			  // 使用单引号或双引号
			  var s1 = 'Hello, Dart!';
			  var s2 = "你好，Dart！";
			  
			  // 字符串拼接
			  var s3 = s1 + ' ' + s2;
			  print(s3); // 输出: Hello, Dart! 你好，Dart！
			  
			  // 字符串插值
			  var name = '张三';
			  var score = 95;
			  var message = '学生 ${name} 的分数是 $score。'; // ${name} 的 {} 可以省略
			  print(message); // 输出: 学生 张三 的分数是 95。
			  
			  // 多行字符串
			  var multiLineString = """
			  这是一个
			  可以跨行的
			  字符串。
			  """;
			  print(multiLineString);
			  
			  // 原始字符串
			  var rawString = r'在原始字符串中，\n 不会作为换行符处理。';
			  print(rawString); // 输出: 在原始字符串中，\n 不会作为换行符处理。
			  ```
	- **布尔类型**
		- `bool` 只有两个可能的值：`true` 和 `false`。
		- 与某些语言不同，Dart 是强布尔类型的，这意味着只有 `true` 值被认为是真。像 `1`、`'hello'`、`[]` 等非空值不会被隐式转换成 `true`。
		- **代码示例：**
			- ```dart
			  var isStudent = true;
			  bool hasPassed = false;
			  
			  if (isStudent) {
			    print('是学生');
			  }
			  
			  // 错误的用法，这在 Dart 中会报错
			  // if (1) { ... }
			  ```
	- **Null**
		- Dart 中有一个特殊的类型叫 `Null`，它只有一个值，就是 `null`，表示没有值或为空。
		- 默认情况下，所有类型都是不可为空的，也就是说，不能把 `null` 赋值给一个 `String` 类型的变量，除非你明确指定它可以为空。
		- 如果想声明一个可为空的类型，只需在类型名后加上问号 `?`。例如，`String?` 表示这个变量可以是一个字符串，也可以是 `null`。
- 集合类型
  heading:: true
	- `List`
		- 是一个有序的对象集合，可以包含重复元素。
		- 类似于 Java 的 `ArrayList`。
		- **代码示例：**
			- ```dart
			  // 使用字面量创建 List (推荐)
			  var numbers = [1, 2, 3, 4, 5]; // 推断为 List<int>
			  var mixed = [1, 'hello', true]; // 推断为 List<Object>
			  
			  // 显式声明类型
			  List<String> names = ['Alice', 'Bob', 'Charlie'];
			  
			  // 访问元素（索引从 0 开始）
			  print(names[0]); // 输出: Alice
			  
			  // 添加元素
			  names.add('David');
			  
			  // 获取长度
			  print(names.length); // 输出: 4
			  
			  // 遍历
			  for (var name in names) {
			    print(name);
			  }
			  
			  names.forEach((name) => print(name));
			  
			  // 创建固定长度或可增长的列表
			  var growableList = <String>[]; // 可增长列表
			  growableList.add('a');
			  ```
	- `Set`
		- 是一个无序的、元素唯一的对象集合。
		- 类似于 Java 的 `HashSet`。
		- **代码示例：**
			- ```dart
			  // 使用字面量创建 Set
			  var fruits = {'apple', 'banana', 'orange'}; // 推断为 Set<String>
			  fruits.add('apple'); // 添加重复元素无效
			  print(fruits); // 输出: {apple, banana, orange}
			  
			  // 创建一个空 Set
			  var emptySet = <String>{};
			  // 注意: var empty = {}; 这样会创建一个 Map，而不是 Set
			  
			  // 检查是否包含元素
			  print(fruits.contains('banana')); // 输出: true
			  
			  // 集合操作
			  var setA = {1, 2, 3};
			  var setB = {3, 4, 5};
			  print(setA.intersection(setB)); // 交集: {3}
			  print(setA.union(setB));        // 并集: {1, 2, 3, 4, 5}
			  ```
	- `Map`
		- 是一个键值对的集合。键是唯一的，每个键关联一个值。
		- 类似于 Java 的 `HashMap`。
		- **代码示例：**
			- ```dart
			  // 使用字面量创建 Map
			  var student = {
			    'name': 'Tom',
			    'age': 20,
			    'major': 'Computer Science'
			  }; // 推断为 Map<String, Object>
			  
			  // 显式声明类型
			  var scores = <String, int>{
			    'Math': 95,
			    'English': 88
			  };
			  
			  // 访问值
			  print(student['name']); // 输出: Tom
			  
			  // 添加新的键值对
			  scores['Physics'] = 92;
			  
			  // 获取所有键或所有值
			  print(scores.keys);   // 输出: (Math, English, Physics)
			  print(scores.values); // 输出: (95, 88, 92)
			  
			  // 检查键是否存在
			  print(scores.containsKey('Math')); // 输出: true
			  ```
- 特殊类型
  heading:: true
	- `Object`
		- Dart 中所有非 `null` 类型的根类是 `Object`（不包括 `Null` 类型）。
		- `Object` 类型的变量可以存储任何值，但默认情况下你只能调用 `Object` 类本身提供的方法，除非先将它转换成更具体的类型。
	- `dynamic`
		- `dynamic` 类型会告诉编译器关闭类型检查，你可以将任何类型的值赋给 `dynamic` 变量，并调用它的任意方法。如果调用的方法不存在，错误会在运行时而不是编译时抛出。
	- `Record`
		- `Record` 是一种匿名的、不可变的复合类型，用于将多个不同类型的值打包在一起。
		- **代码示例：**
			- ```dart
			  // 创建一个 Record
			  var record = ('John', 25, 'USA');
			  print(record.$1); // 按位置访问，索引从 1 开始。输出: John
			  print(record.$2); // 输出: 25
			  
			  // 创建带命名字段的 Record
			  var namedRecord = (name: 'Jane', age: 30, country: 'Canada');
			  print(namedRecord.name); // 按名字访问。输出: Jane
			  
			  // 函数可以返回一个 Record，包含多个值，用于临时聚合数据进行传递，无需定义类。
			  (String, int) getUserInfo() {
			    return ('Alice', 18);
			  }
			  
			  var userInfo = getUserInfo();
			  print('Name: ${userInfo.$1}, Age: ${userInfo.$2}');
			  
			  // 使用模式匹配解构 Record
			  var (name, age) = getUserInfo();
			  print('Name: $name, Age: $age');
			  ```
	- `void`
		- 在 Dart 中，`void` 是一个内建类型的名称，用于表示函数没有返回值、不需要关心返回值。
-