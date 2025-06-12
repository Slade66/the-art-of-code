- 一个类只做一件事。
- 如果一个类承担了过多功能，任何一个功能的修改都可能影响到其它无关功能，导致代码难以维护。
- **比喻：**想象一把瑞士军刀。它既能当刀，又能当螺丝刀，还能当开瓶器。这很酷，但如果它的螺丝刀部分坏了，你可能需要修理或更换整把军刀，这会影响到你使用它的其他功能。单一职责原则告诉我们，最好让一把刀只负责切割，一个螺丝刀只负责拧螺丝。各司其职，一个坏了不影响另一个。
- **违反单一职责原则的示例：**
	- ```java
	  // User 类同时负责表示用户数据和处理数据库操作，职责不单一。
	  // 它可能因用户结构变化或数据库逻辑变化而被修改。
	  public class User {
	      private int id;
	      private String name;
	  
	      public User(int id, String name) {
	          this.id = id;
	          this.name = name;
	      }
	  
	      public void saveToDatabase() {
	          // ... 连接数据库，执行插入或更新 ...
	          System.out.println("Saving user " + name + " to database");
	      }
	  }
	  ```
- **遵循单一职责原则的示例：**
	- ```java
	  // User 类仅用于承载用户数据。
	  // 它的变更只可能因为用户属性的变化。
	  public class User {
	      private int id;
	      private String name;
	  
	      public User(int id, String name) {
	          this.id = id;
	          this.name = name;
	      }
	  
	      // getters and setters...
	  }
	  
	  // UserRepository 类专注于用户数据的持久化处理。
	  // 它的变更只会受到存储逻辑或数据库变化的影响。
	  public class UserRepository {
	      // 可能包含数据库连接等资源
	  
	      public void save(User user) {
	          // ... 连接数据库，执行插入或更新 ...
	          System.out.println("Saving user " + user.getName() + " to database");
	      }
	  }
	  
	  ```
-