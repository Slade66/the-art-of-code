- 在 Dart 中，构造方法是一个与类同名的特殊函数，用于创建（或初始化）类的实例。它的核心任务是初始化实例的成员变量。
- **默认构造方法：**
	- 如果你没有显式声明任何构造方法，Dart 会自动为你生成一个无参的默认构造方法，并且它会调用父类的无参构造方法。
	- 一旦你声明了任意一个构造方法，这个默认构造方法就不会再自动生成。
	- **作用：**减少代码冗余。如果你只需要一个不带参数的构造过程，就无需手动编写，Dart 会自动生成，帮你省去这部分代码。这对于一些不需要初始化参数的类来说非常实用。
	- 这一行为与 Java 完全一致。
- **标准构造方法：**
	- 创建一个新的类实例并初始化其字段。
	- ```dart
	  class Point {
	    double x;
	    double y;
	  
	    // 标准构造方法
	    Point(double x, double y) {
	      this.x = x;
	      this.y = y;
	    }
	  }
	  
	  void main() {
	    var p = Point(10, 20);
	    print(p.x); // 输出: 10
	  }
	  ```
- **用于快速初始化成员变量的语法糖：**
	- 用于简化常见的“参数赋值给成员变量”的模式。
	- 在大多数语言（如 Java）中，参数列表只能列出“值”，不能进行赋值操作，因此必须在构造方法体内显式写出 `this.x = x;`。而 Dart 将赋值逻辑前移到参数列表中。
	- 可以直接在构造方法的参数列表中使用 `this.` 来初始化同名的成员变量，编译器会自动生成对应的赋值语句，从而省去大量手动编写的样板代码。
	- **只能初始化同名成员变量：**写 `this.name` 时，Dart 会自动识别并初始化名为 `name` 的成员变量，否则会报错。
	- **不能包含逻辑，只能用于赋值：**如果需要额外的处理逻辑，就必须使用传统的构造函数体来完成。
	- ```dart
	  class Point {
	    double x;
	    double y;
	  
	    // 使用初始化形式参数，效果同上，但代码更简洁
	    Point(this.x, this.y);
	  }
	  
	  void main() {
	    var p = Point(10, 20);
	    print(p.y); // 输出: 20
	  }
	  ```
- **初始化列表：**
	- 初始化列表是构造函数体执行之前，用来初始化成员变量的特殊语法块。
	- 它出现在构造函数的参数列表和函数体之间，用冒号 `:` 开始，可以包含一个或多个初始化语句。
	- 一旦你要初始化 `final` 成员变量或调用 `super(...)`，就必须使用初始化列表。
	- **语法：**
		- ```dart
		  ClassName(parameters) : property1 = value1, property2 = value2 {
		    // 构造函数体
		  }
		  ```
	- **注意：**
		- 只能用于成员变量赋值，不能执行复杂语句。
		- 它们在构造函数体之前执行。
		- 初始化的变量必须是成员变量。
		- 初始化时可以执行简单的表达式。
- **命名构造方法：**
	- 一个类可以有多个生成构造方法，只要它们的名字不同。这通过 `ClassName.identifier` 的形式实现。
	- 这在功能上类似于 Java 的构造方法重载。在 Java 中，你通过不同的参数列表来区分构造方法。在 Dart 中，你可以为不同的构造方法赋予清晰的、有意义的名字，代码可读性更高。
	- **代码示例：**
		- ```dart
		  class JsonData {
		    String data;
		  
		    // 默认构造方法
		    JsonData(this.data);
		  
		    // 命名构造方法：从一个 Map 创建实例
		    JsonData.fromMap(Map<String, dynamic> map)
		        : data = map.toString() { // 使用初始化列表
		      print('通过 Map 创建实例');
		    }
		  
		    // 命名构造方法：加载一个空实例
		    JsonData.empty() : data = '{}';
		  }
		  
		  void main() {
		    var d1 = JsonData('{"name":"Alice"}'); // 使用默认构造方法
		    var d2 = JsonData.fromMap({'name': 'Bob'}); // 使用命名构造方法
		    var d3 = JsonData.empty(); // 使用另一个命名构造方法
		  
		    print(d1.data); // 输出: {"name":"Alice"}
		    print(d2.data); // 输出: {name: Bob}
		    print(d3.data); // 输出: {}
		  }
		  ```
- **重定向构造方法：**
	- 有时，一个构造方法可能只是想调用同一个类中的另一个构造方法。这可以通过重定向构造方法实现。这种构造方法没有方法体，它使用 `this` 关键字（而不是类名）来调用另一个构造方法。
	- **代码示例：**
		- ```dart
		  class Car {
		    String make;
		    String model;
		    int year;
		  
		    // 主要的构造方法
		    Car(this.make, this.model, this.year);
		  
		    // 重定向构造方法：创建一个特定年份的 "Ford"
		    // 它没有方法体，直接将任务委托给了默认构造方法
		    Car.ford(String model, int year) : this('Ford', model, year);
		  }
		  
		  void main() {
		    var myCar = Car.ford('Mustang', 2025);
		    print('${myCar.make} ${myCar.model} ${myCar.year}'); // 输出: Ford Mustang 2025
		  }
		  ```
- **常量构造方法：**
	- 如果你的类用于创建不可变对象（即所有字段都是 `final`），可以定义一个以 `const` 关键字开头的构造方法。这样，类的实例就可以在编译时作为常量创建，从而带来性能上的优势：
		- **内存复用：**当你使用 `const` 构造方法创建对象时，只要构造参数相同，Dart 不会重复创建多个对象实例，而是复用已经存在的常量对象。
		- **避免运行时初始化：**使用 `const` 对象时，实例在编译期就已创建，无需在运行时再次执行构造方法。这在 UI 框架（如 Flutter）中尤为重要，因为组件可能频繁重建，使用 `const` 可显著减少重复计算和内存分配。
	- **代码示例：**
		- ```dart
		  class ImmutablePoint {
		    final double x;
		    final double y;
		  
		    // 常量构造方法
		    const ImmutablePoint(this.x, this.y);
		  }
		  
		  void main() {
		    // 使用 const 构造方法创建编译时常量
		    const p1 = ImmutablePoint(1, 1);
		    const p2 = ImmutablePoint(1, 1);
		  
		    // 因为 p1 和 p2 都是编译时常量并且值相同，
		    // Dart 不会创建新对象，而是复用同一个实例。
		    print(identical(p1, p2)); // 输出: true
		  
		    // 如果不用 const 关键字，即使值相同，也会创建新对象
		    var p3 = ImmutablePoint(1, 1);
		    var p4 = ImmutablePoint(1, 1);
		    print(identical(p3, p4)); // 输出: false
		  }
		  ```
-