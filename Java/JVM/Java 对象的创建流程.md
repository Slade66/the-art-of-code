---
创建日期: 2025-04-18
---

下面以使用 `new` 关键字创建对象的流程为例进行说明。

#### 步骤一：触发类加载（如果需要）

当 JVM 执行引擎遇到一条用于创建类实例的 `new` 字节码指令时，会首先检查类是否已经完成加载、链接和初始化。如果尚未完成，JVM 会立即触发类加载过程，且该过程必须成功，否则将无法继续创建对象；如果类已经完成了这三个阶段，则可以直接进入下一步。

#### 步骤二：堆内存分配

一旦确认类已经就绪，JVM 就会在 Java 堆上为新对象实例分配内存。

分配内存的第一步是确定对象所需的大小，主要包括三部分：
1. 对象头的开销；
2. 实例变量（非静态字段）所占的空间，包括当前类定义的字段和从父类继承的字段；
3. 为满足内存对齐要求而添加的填充字节（如补齐到 8 字节的倍数）。

确定大小后，JVM 会在堆中寻找一块足够大且连续的空闲内存。常见的内存分配策略有两种：
- 指针碰撞（Bump-the-Pointer）：已用和未用内存通过一个指针分隔，分配时只需将该指针向后移动对象所需的大小即可。
- 空闲列表（Free List）：JVM 维护一个空闲内存块列表，分配时从中找出一块合适的区域并进行更新。

为了避免多线程在堆上分配内存时因竞争共享指针而产生的同步开销，JVM 通常会为每个线程在 Eden 区预先分配一小块私有内存，称为 TLAB（Thread Local Allocation Buffer）。线程在创建对象时优先在自己的 TLAB 中进行分配，因为 TLAB 是线程私有的，无需加锁，分配效率极高。只有当对象过大无法放入 TLAB，或 TLAB 空间耗尽时，才会回退到共享堆内存中分配，此时可能涉及加锁。

#### 步骤三：执行一系列初始化步骤

**3.1 内存零化**

在对象内存分配完成后，JVM 会对新分配的内存空间（不包括对象头）执行内存零化操作，即将该区域内的所有字节清零。这意味着，所有基本类型字段会被设置为 `0`、`0.0` 或 `false`，引用类型字段则被设置为 `null`。

内存零化不仅是 Java 中对象实例字段拥有默认值（如 `int` 为 `0`，`boolean` 为 `false`，引用为 `null`）的根本机制，也具有重要的实用价值：它既能防止程序误读堆中旧对象遗留的数据，避免出现不可预测的错误，又能阻止前一个对象的敏感信息被新对象无意访问，从而提升系统的安全性和稳定性。

**3.2 设置对象头**

JVM 会对对象头进行必要的设置，包括指向对象所属类的元数据（Klass Pointer），以及初始化 Mark Word 的状态（如锁状态为无锁，GC 年龄为 0 等）。

**3.3 初始化父类**

Java 规定，在初始化子类之前，必须先初始化父类。这是因为子类继承了父类的成员（字段和方法），而子类的后续操作通常依赖于父类的状态已准备就绪。就像盖房子一样，必须先完成父类提供的“基础结构”，子类才能在其上添加“特色功能”。Java 通过构造函数链机制强制确保这一顺序：每个子类构造函数必须首先调用父类构造函数，完成父类的初始化，从而确保子类在调用继承的方法时（这些方法可能依赖父类成员），父类的“地基”已经稳固可用。

**3.4 实例变量初始化与实例初始化块执行**

在 Java 中，实例变量的显式初始化（如 `int count = 1;`）和实例初始化块（即不带 `static` 的 `{...}` 代码块）会严格按照它们在源代码中出现的顺序执行。

实际上，`javac` 编译器会将这些初始化代码收集并插入到每个构造函数的字节码中。这些初始化指令并非位于构造函数的开头，而是紧跟在父类构造函数 `super()`（或当前类的构造函数 `this()`）调用之后，并在构造函数体中的其它代码执行之前。

**3.5 构造函数执行**

JVM 根据提供的参数列表选择并调用匹配的构造函数。在父类构造和当前类的实例初始化（包括变量赋值和初始化块）完成后，构造函数体内的代码才开始执行。这部分代码使用传入的参数进一步设定对象的初始状态，因此，它赋予成员变量的值可能会覆盖实例初始化阶段赋予的初始值。

### 步骤四：返回引用

当构造函数执行完毕并正常返回时，`new` 表达式的结果是一个指向刚刚在堆上创建并初始化好的对象的引用（本质上是该对象的内存地址）。这个引用随后会被赋给 `new` 表达式左侧的变量。

如果在构造函数执行过程中发生异常，JVM 会停止当前构造函数的执行并进行异常处理。此时，已分配的内存（即对象）会被自动回收，构造函数不会正常返回，因此 `new` 表达式不会返回有效的对象引用。如果是局部变量，且在声明时没有初始化，而构造过程中抛出异常，局部变量将不会获得任何值，也不会是 `null`。
