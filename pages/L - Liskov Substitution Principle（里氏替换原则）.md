- 子类应当能够替代父类，并在使用中保持程序行为一致，不能引发错误或逻辑混乱。
- 避免那种“看起来是子类，行为却完全不同”的继承方式。
- 在编写子类时，务必思考：“它真的属于这种类型吗？”
- 如果你发现必须大幅修改或重写父类的方法，才能让子类正常工作，那么就应该考虑放弃继承，改用组合等其他设计方式。
- 确保你的实现没有“惊喜”。
- **违反里氏替换原则的示例：**
	- ```java
	  class Rectangle {
	      protected int width;
	      protected int height;
	  
	      public void setWidth(int w) {
	          width = w;
	      }
	  
	      public void setHeight(int h) {
	          height = h;
	      }
	  
	      public int getArea() {
	          return width * height;
	      }
	  }
	  
	  class Square extends Rectangle {
	      @Override
	      public void setWidth(int w) {
	          width = height = w; // 强制保持宽高一致
	      }
	  
	      @Override
	      public void setHeight(int h) {
	          width = height = h; // 强制保持宽高一致
	      }
	  }
	  
	  // 假设有一个用于调整尺寸的函数：
	  public void resize(Rectangle r) {
	      r.setWidth(5);
	      r.setHeight(10);
	      System.out.println("Area: " + r.getArea());
	  }
	  
	  // 运行结果：
	  Rectangle r1 = new Rectangle();
	  resize(r1);  // 输出: Area: 50 ✅
	  
	  Rectangle r2 = new Square();
	  resize(r2);  // 输出: Area: 100 ❌
	  ```
	- 把子类（Square）当作父类（Rectangle）用时，行为变了，程序出错。
-