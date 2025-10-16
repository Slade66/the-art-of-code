- 使用小而专的接口，而不是大而全的接口。
- 客户端不应被迫实现不需要的方法，应通过拆分接口，提供多个职责单一的专用接口。
- 拆分接口有助于降低模块间的耦合度：当客户端仅依赖其所需的方法时，接口其他部分的变动不会对其产生影响。
- **不良示例：大而全的接口**
	- ```java
	  public interface Worker {
	      void work();
	      void eat();
	      void sleep();
	  }
	  
	  public class Robot implements Worker {
	      @Override
	      public void work() {
	          System.out.println("Robot working.");
	      }
	  
	      @Override
	      public void eat() {
	          // Robot doesn't eat
	          throw new UnsupportedOperationException();
	      }
	  
	      @Override
	      public void sleep() {
	          // Robot doesn't sleep
	          throw new UnsupportedOperationException();
	      }
	  }
	  ```
	- **问题：**`Robot` 不需要 `eat()` 和 `sleep()` 方法，却被强迫实现它们，违反了接口隔离原则，增加了耦合和维护成本。
- **良好示例：拆分为小而专的接口**
	- ```java
	  public interface Workable {
	      void work();
	  }
	  
	  public interface Eatable {
	      void eat();
	  }
	  
	  public interface Sleepable {
	      void sleep();
	  }
	  
	  public class Human implements Workable, Eatable, Sleepable {
	      @Override
	      public void work() {
	          System.out.println("Human working.");
	      }
	  
	      @Override
	      public void eat() {
	          System.out.println("Human eating.");
	      }
	  
	      @Override
	      public void sleep() {
	          System.out.println("Human sleeping.");
	      }
	  }
	  
	  public class Robot implements Workable {
	      @Override
	      public void work() {
	          System.out.println("Robot working.");
	      }
	  }
	  ```
	- **优点：**每个类只依赖自己真正需要的接口，接口变更对不相关的客户端无影响。
-