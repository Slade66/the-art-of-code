- Dart 是一门纯粹的面向对象编程语言，这意味着在 Dart 中，一切都是对象。每个对象都是某个类的实例，而所有类（除了 `Null`）最终都继承自顶层类 `Object`。甚至数字、函数，乃至 `null` 本身，也都被视为对象。
- 类与对象
  heading:: true
	- 对象是一种用于存储数据的数据结构，可以看作是数据的容器。
	- 类是创建对象的蓝图，它定义了对象的结构，规定了对象中应包含哪些类型的数据，并封装了用于操作这些数据的方法。
	- `super` 关键字是对一个类的直接父类的引用。通过它，你可以在子类中访问父类的成员。
		- 两个核心用途：
			- 调用父类的构造方法。
				- 调用父类构造方法必须在初始化列表中完成，也就是在子类构造方法体 `{}` 执行之前。
					- ```dart
					  // 父类 Person
					  class Person {
					    String name;
					    int age;
					  
					    // Person 的构造方法
					    Person(this.name, this.age) {
					      print('Person constructor executed.');
					    }
					  
					    void display() {
					      print('Name: $name, Age: $age');
					    }
					  }
					  
					  // 子类 Student
					  class Student extends Person {
					    String major;
					  
					    // Student 的构造方法
					    // 在初始化列表中使用 : super(...) 来调用父类的构造方法
					    Student(String name, int age, this.major) : super(name, age) {
					      print('Student constructor executed.');
					    }
					  }
					  
					  void main() {
					    var student = Student('小明', 20, 'Computer Science');
					    student.display();
					  }
					  ```
				- 如果子类构造方法的参数只是为了直接传递给父类构造方法，你可以使用 `super.` 前缀。
					- ```dart
					  class Student extends Person {
					    String major;
					  
					    // 使用 super. 来声明参数并自动传递给父类构造方法
					    // super.name 意味着：
					    // 1. 声明一个名为 name 的参数。
					    // 2. 将接收到的这个参数值，自动传递给父类构造方法的 name 参数。
					    Student(super.name, super.age, this.major) {
					      print('Student constructor executed.');
					    }
					  }
					  
					  void main() {
					    var student = Student('小红', 19, 'Physics');
					    student.display();
					  }
					  ```
			- 调用父类的方法或属性。当子类重写（`@override`）了父类的某个方法时，有时你需要在子类的新实现中，依然执行父类版本的逻辑。这时就需要用 `super` 来显式调用父类的方法。
	- Dart 中使用 `_` 前缀表示私有变量，其作用域仅限于当前库。在 Dart 中，一个 `.dart` 文件就是一个库，因此这类变量只能在同一个文件内访问。这与 Java 中基于类的 `private` 修饰符不同，需要将思路从“类级别的封装”转变为“文件级别的封装”。
	- 在 Dart 中创建对象的方式类似于 Java，可以使用 `new` 关键字或直接调用构造函数，更推荐省略 `new` 的写法。
	- Dart 使用 `get` 和 `set` 关键字定义 getter 和 setter，让你写出“属性一样的代码”，但背后仍然是方法。
		- Getter 和 Setter 必须和字段名一样。
		- Getter 有返回值，Setter 没有返回值（`void` 类型）。
		- Getter 语法：
			- ```dart
			  Type get propertyName {
			    // 方法体，返回一个值
			  }
			  
			  String get name {
			    return _name;
			  } // 适合需要多行语句、逻辑更复杂的情况
			  
			  String get name => _name; // 箭头函数形式
			  ```
		- Setter 语法：
			- Setter 不能用箭头函数，因为它不返回值、需要执行语句块。
			- ```dart
			  set propertyName(Type value) {
			    // 方法体，接收一个值
			  }
			  
			  set name(String value) {
			    if (value.isEmpty) {
			      throw ArgumentError('Name cannot be empty');
			    }
			    _name = value;
			  }
			  ```
	- **代码示例：**
		- ```dart
		  // 定义一个 'Clerk' 类
		  class Clerk {
		    // 实例变量 (属性)
		    String name;
		    final String employeeID; // final 属性只能赋值一次
		    int _age; // 以下划线 (_) 开头，表示私有属性
		  
		    // 构造方法 (使用语法糖和初始化列表)
		    Clerk(this.name, String id, int age)
		        : employeeID = id,
		          _age = age;
		  
		    // 实例方法 (行为)
		    void sayHello() {
		      print('大家好，我是 $name，工号是 $employeeID。');
		    }
		  
		    // Getter: 提供对私有属性的只读访问
		    int get age => _age;
		  
		    // Setter: 提供对私有属性的写访问，可以加入逻辑
		    set age(int value) {
		      if (value > 0) {
		        _age = value;
		      }
		    }
		  }
		  
		  void main() {
		    // 创建 Clerk 类的实例 (对象)
		    var clerk1 = Clerk('张三', 'EMP-001', 25);
		    
		    // 访问属性和调用方法
		    print(clerk1.name); // 张三
		    clerk1.sayHello(); // 大家好，我是 张三，工号是 EMP-001。
		  
		    // 使用 getter 和 setter
		    print(clerk1.age); // 25 (通过 getter)
		    clerk1.age = 26;   // (通过 setter)
		    print(clerk1.age); // 26
		  }
		  ```
	- [[Dart 的构造方法]]
- 继承
  heading:: true
	- 继承允许一个类（子类）获取另一个类（父类）的属性和方法。这实现了代码复用。
	- **关键字：**使用 `extends` 关键字，与 Java 相同。
	- **单继承：**Dart 和 Java 一样，只支持单继承，即一个类只能有一个直接父类。
	- 使用 `super` 可以调用父类的构造方法或其它方法。在 Dart 中，调用父类构造方法 `super()` 必须写在子类构造方法的初始化列表中，而不像 Java 那样放在方法体的第一行。
	- **代码示例：**
		- ```dart
		  // 父类
		  class Vehicle {
		    String brand;
		    int year;
		  
		    Vehicle(this.brand, this.year);
		  
		    void move() {
		      print('$brand is moving.');
		    }
		  }
		  
		  // 子类继承父类
		  class Car extends Vehicle {
		    String model;
		  
		    // 调用父类的构造方法必须在初始化列表中使用 super
		    Car(String brand, this.model, int year) : super(brand, year);
		  
		    // 重写父类的方法
		    @override
		    void move() {
		      super.move(); // 可以选择性地调用父类的实现
		      print('The car $model is driving on the road.');
		    }
		  }
		  
		  void main() {
		    var myCar = Car('Ford', 'Mustang', 2025);
		    myCar.move();
		    // 输出:
		    // Ford is moving.
		    // The car Mustang is driving on the road.
		  }
		  ```
- 抽象类
  heading:: true
	- 抽象类不能被实例化，通常用作基类，定义通用接口和部分实现。
	- **关键字：**使用 `abstract` 关键字，与 Java 完全一样。
	- **抽象方法：**只有声明没有实现的方法，子类必须实现它。
	- **示例代码：**
		- ```dart
		  abstract class Animal {
		    // 抽象方法，没有方法体
		    void makeSound();
		  
		    // 普通方法，提供默认实现
		    void sleep() {
		      print('Sleeping...');
		    }
		  }
		  
		  class Dog extends Animal {
		    @override
		    void makeSound() {
		      print('Woof! Woof!');
		    }
		  }
		  
		  void main() {
		    // var animal = Animal(); // 错误：抽象类不能被实例化
		    var dog = Dog();
		    dog.makeSound(); // Woof! Woof!
		    dog.sleep();     // Sleeping...
		  }
		  ```
- 接口
  heading:: true
	- Dart 没有单独的 `interface` 关键字，任意类都可作为接口用，每个类默认都会隐式定义一个接口，包含该类所有的公共实例成员（方法和属性）。
	- 当一个类需要遵循某个接口时，使用 `implements` 关键字。`implements` 一个类时，必须实现该接口中定义的所有方法。一个类可以实现多个接口。
	- Dart 中的 `implements` 不会继承任何已有的实现（包括字段），它只继承接口的结构约定，也就是说你只继承接口的定义，而不继承具体的实现代码。
	- Dart 的多态和 Java 的多态完全相同。
	- 类中所有的公共实例变量会被视为包含对应的 getter 和 setter。因此，在实现接口时，需要手动声明这些变量，并提供相应的 getter 和 setter 实现。
		- ```dart
		  class Person {
		    String name = '';
		  }
		  
		  class Student implements Person {
		    String _name = ''; // ✅ 你必须自己定义变量
		  
		    @override
		    String get name => _name;
		  
		    @override
		    set name(String value) => _name = value;
		  }
		  ```
	- **示例代码：**
		- ```dart
		  // Person 类隐式地定义了一个接口，包含 name 和 greet()
		  class Person {
		    final String name;
		    Person(this.name);
		  
		    String greet() {
		      return 'Hello, my name is $name.';
		    }
		  }
		  
		  // Student 类实现了 Person 接口
		  // 它必须提供自己的 name 和 greet() 实现
		  class Student implements Person {
		    @override
		    String name; // 必须重新声明属性
		  
		    String major;
		  
		    Student(this.name, this.major);
		    
		    @override
		    String greet() { // 必须重新实现方法
		      return 'Hi, I am $name, a student of $major.';
		    }
		  }
		  
		  void main() {
		    var student = Student('小明', 'Computer Science');
		    print(student.greet()); // Hi, I am 小明, a student of Computer Science.
		  }
		  ```
- Mixin
  heading:: true
	- Mixin 是一种在多个类层次结构中复用代码的方式，可以看作是“带实现的接口”。当你发现不同继承链上的类具有相同的行为时，使用 Mixin 是一个理想的选择。
	- **关键字：**使用 `mixin` 来定义一个混入，使用 `with` 来应用它。
	- **示例代码：**
		- ```dart
		  // 定义一个混入，提供日志功能
		  mixin Logger {
		    void log(String message) {
		      print('${DateTime.now()}: $message');
		    }
		  }
		  
		  class Service {
		    void performAction() {
		       // ...
		    }
		  }
		  
		  // Manager 类继承自 Service，并混入了 Logger 的功能
		  class Manager extends Service with Logger {
		    String name;
		    Manager(this.name);
		  
		    void manage() {
		      log('Manager $name is managing tasks.'); //可以直接使用 log 方法
		      performAction();
		    }
		  }
		  
		  class Controller with Logger { // 也可以混入到一个没有父类的类
		    void process() {
		      log('Controller is processing request.');
		    }
		  }
		  
		  void main() {
		    var manager = Manager('Alice');
		    manager.manage();
		  
		    var controller = Controller();
		    controller.process();
		  }
		  ```
-