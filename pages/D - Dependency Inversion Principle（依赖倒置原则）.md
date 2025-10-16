- 依赖于抽象，而非具体实现。
- 面向接口编程，而不是面向实现编程。
- 高层模块（如业务逻辑）不应直接依赖底层模块（如数据库实现），而应通过抽象层（接口或抽象类）进行交互。
- 调用方不依赖被调用方的具体实现，因此在更换底层实现时，无需修改调用代码。
- 这是实现解耦的关键，能使系统的某一部分（例如数据库）可以被轻松替换，而不会波及其他模块。例如，可以将 MySQL 替换为 PostgreSQL，而无需修改业务代码。
- **比喻：**你的电脑不依赖某个特定品牌的键盘，而是依赖于标准的 USB 接口（抽象）。任何符合 USB 标准的键盘（实现）都能接入电脑正常使用。这使你可以自由更换键盘，而无需更改电脑本身。
- **代码示例：**
	- ```java
	  // 抽象接口：定义高层模块依赖的标准
	  interface Database {
	      void save(String data);
	  }
	  
	  // 具体实现1：MySQL数据库
	  class MySQLDatabase implements Database {
	      @Override
	      public void save(String data) {
	          System.out.println("保存到 MySQL：" + data);
	      }
	  }
	  
	  // 具体实现2：MongoDB数据库
	  class MongoDBDatabase implements Database {
	      @Override
	      public void save(String data) {
	          System.out.println("保存到 MongoDB：" + data);
	      }
	  }
	  
	  // 高层模块：业务逻辑类，依赖于 Database 接口
	  class DataManager {
	      private Database database;
	  
	      // 构造方法注入依赖（依赖注入的一种方式）
	      public DataManager(Database database) {
	          this.database = database;
	      }
	  
	      public void saveData(String data) {
	          database.save(data);
	      }
	  }
	  
	  // 程序入口
	  public class Main {
	      public static void main(String[] args) {
	          // 可随意更换底层实现而不影响高层逻辑
	          Database db = new MySQLDatabase(); // 也可以替换为 new MongoDBDatabase();
	          DataManager manager = new DataManager(db);
	  
	          manager.saveData("用户数据");
	      }
	  }
	  ```
-