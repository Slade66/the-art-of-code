- **什么是多对一？**
	- 在 GORM 中，“多对一” 关系通常被称为 Belongs To（属于）。
	- 想象一下员工（User）和公司（Company）的关系：
		- 一个公司有多个员工。
		- 一个员工只属于（Belongs To）一个公司。
		- 从员工的角度看，这就是“多对一”关系。
	- 在数据库层面，这意味着员工表中必须有一个列用来存储公司表的主键。这个列就是外键。
	- 在 GORM 中实现 Belongs To 关系，关键在于在“多”的一方（子表）中保存“一”的一方（父表）的主键作为外键。谁拥有外键，谁就 "Belongs To"（属于） 谁。
	- ```go
	  package main
	  
	  import "gorm.io/gorm"
	  
	  // Company 公司表（主表）
	  type Company struct {
	      ID   uint
	      Name string
	  }
	  
	  // User 用户表（从表）
	  // User 属于 Company
	  type User struct {
	      gorm.Model
	      Name string
	  
	      // 1. 外键 (Foreign Key)：通常是 <关联表名>ID
	      CompanyID int 
	      
	      // 2. 关联字段 (Association Field)：用来存放查询出来的 Company 结构体
	      Company   Company 
	  }
	  ```
	- **外键字段：**
		- 通常命名为：`[关联结构体名]ID`
		- 这是数据库中实际存储的列（外键）。
	- **关联结构体字段：**
		- 用于存储查询到的 Company 对象，默认情况下它不会在数据库员工表中生成列，但在查询时，GORM 会把对应的公司数据填充到这里。
- **GORM 的默认约定：**
	- GORM 看到 `User` 结构体中有一个 `Company` 字段。
	- 它会自动寻找名为 `CompanyID` 的字段作为外键。
	- 它默认 `CompanyID` 对应 `Company` 表的 `ID` 字段。
	- **自定义外键：**
		- 如果你的数据库字段命名不规范，比如外键叫 `CompID`，你需要使用 Struct Tag 来告诉 GORM：
			- **`foreignKey`：**指定当前结构体（User）中的哪个字段是外键。
			- **`references`：**指定目标结构体（Company）中的哪个字段被引用（通常默认为 ID，一般不需要写）。
- **GORM 关联字段使用指针或值的区别：**
	- | **特性** | **指针类型 (*Company)** | **值类型 (Company)** |
	  | **空值表现** | `nil` (真·空) | **零值** (ID为0, 字段为空串) |
	  | **JSON 输出** | `"Company": null` | `"Company": { "ID":0, ... }` |
	  | **代码安全性** | ⚠️ **低** (直接调用会 Panic，需判空) | ✅ **高** (永远不会崩，安全访问) |
	  | **语义表达** | "没有公司" | "有一个空的公司对象" |
	- 如果关联字段是指针，外键 ID 也用指针 (`CompanyID *int`)，这样数据库可以存 SQL `NULL`。
- **代码示例：**
	- **创建数据：**
		- **创建用户时，同时创建一个新公司：**
			- ```go
			  user := User{
			      Name: "张三",
			      Company: Company{
			          Name: "字节跳动", // GORM 会先创建这个公司，拿到 ID，填入 User 的 CompanyID，再创建 User
			      },
			  }
			  
			  result := db.Create(&user) // 此时 user.CompanyID 会被自动填充
			  ```
		- **把用户分配给已存在的公司：**
			- ```go
			  var company Company
			  db.First(&company, 1) // 假设找到了 ID 为 1 的公司
			  
			  user := User{
			      Name:      "李四",
			      CompanyID: int(company.ID), // 直接赋值外键
			  }
			  
			  db.Create(&user)
			  ```
	- **查询数据：**
		- 如果你直接查询 User，GORM 默认不会把关联的 Company 查出来（为了性能）。
			- ```go
			  var user User
			  db.First(&user, 1)
			  
			  // 此时 user.CompanyID 有值，但 user.Company 是空的结构体！
			  // user.Company.Name 是空字符串
			  ```
		- **使用 Preload（预加载）：**
			- 你需要告诉 GORM：“帮我把 Company 也一起查出来”。
			- ```go
			  var user User
			  // Preload 的参数必须是 User 结构体中的字段名 "Company"
			  db.Preload("Company").First(&user, 1)
			  
			  fmt.Println(user.Name)         // 输出: 张三
			  fmt.Println(user.Company.Name) // 输出: 字节跳动
			  ```
-