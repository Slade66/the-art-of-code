- 对扩展开放，对修改关闭。
- 多用接口，少改代码。
- 在不修改现有稳定代码的基础上增加新功能，可以有效防止引入新的 bug。
- **比喻：**想象你的手机。你可以通过安装新的 App 来扩展功能，但不需要、也不应该为了兼容某个 App 而拆开手机修改电路板。良好的设计应在不改动原有结构的前提下，便于功能扩展。
- **违反开闭原则的示例：**
	- 每新增一种支付方式，都需要修改 `processPayment` 方法，不利于扩展。
	- ```java
	  public class PaymentProcessor {
	      public void processPayment(String method, double amount) {
	          if ("creditcard".equalsIgnoreCase(method)) {
	              System.out.printf("Processing credit card payment of $%.2f%n", amount);
	          } else if ("paypal".equalsIgnoreCase(method)) {
	              System.out.printf("Processing PayPal payment of $%.2f%n", amount);
	          }
	          // 新增支付方式（如支付宝）时必须修改这里，违反开闭原则
	      }
	  }
	  
	  ```
- **遵循开闭原则的示例：**
	- 通过接口实现扩展，新增支付方式时无需修改已有代码，只需新增类。
	- ```java
	  // 1. 定义支付方式的接口（抽象层）
	  public interface PaymentMethod {
	      void pay(double amount);
	  }
	  
	  // 2. 各种支付方式实现该接口（扩展层）
	  public class CreditCard implements PaymentMethod {
	      @Override
	      public void pay(double amount) {
	          System.out.printf("Processing credit card payment of $%.2f%n", amount);
	      }
	  }
	  
	  public class PayPal implements PaymentMethod {
	      @Override
	      public void pay(double amount) {
	          System.out.printf("Processing PayPal payment of $%.2f%n", amount);
	      }
	  }
	  
	  // 新增：支付宝支付，仅新增一个类
	  public class AliPay implements PaymentMethod {
	      @Override
	      public void pay(double amount) {
	          System.out.printf("Processing AliPay payment of ¥%.2f%n", amount * 7.2); // 假设汇率
	      }
	  }
	  
	  // 3. PaymentProcessor 依赖于接口，而非具体实现（对修改关闭，对扩展开放）
	  public class PaymentProcessor {
	      public void process(PaymentMethod method, double amount) {
	          method.pay(amount);
	      }
	  }
	  
	  // 4. 使用示例
	  public class Main {
	      public static void main(String[] args) {
	          PaymentProcessor processor = new PaymentProcessor();
	  
	          processor.process(new CreditCard(), 100.0);
	          processor.process(new PayPal(), 50.0);
	          processor.process(new AliPay(), 80.0); // 新增的支付方式，无需改动旧代码
	      }
	  }
	  ```
-