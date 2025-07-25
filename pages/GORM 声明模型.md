- GORM 通过将 Go 语言的结构体（struct）映射到数据库中的表，使得你可以像操作普通 Go 结构体一样与数据库进行交互。结构体的字段将映射成表的列。
- **示例：**
	- ```go
	  type User struct {
	    ID           uint           // 主键，标准字段
	    Name         string         // 普通字符串字段
	    Email        *string        // 字符串指针，允许为 NULL
	    Age          uint8          // 无符号 8 位整数
	    Birthday     *time.Time     // 时间指针，允许为 NULL
	    MemberNumber sql.NullString // 使用 sql.NullString 处理可为空的字符串
	    ActivatedAt  sql.NullTime   // 使用 sql.NullTime 处理可为空的时间字段
	    CreatedAt    time.Time      // 创建时间，由 GORM 自动填充
	    UpdatedAt    time.Time      // 更新时间，由 GORM 自动填充
	    ignored      string         // 非导出字段不会被 GORM 映射
	  }
	  ```
- **注意：**
	- 指针类型的字段可以为 `null`。
	- 字段名以小写字母开头时，GORM 会忽略该字段，不会映射到数据库。
	- GORM 会将名为 `ID` 的字段作为主键。
	- GORM 会自动将结构体名称转换为小写的复数形式作为表名。例如，`User` 结构体对应到数据库中的 `users` 表，而 `ProductInfo` 则会变成 `product_infos`。
	- 结构体字段名会被自动转换为蛇形命名法（snake_case）。例如，`UserName` 字段会对应到数据库中的 `user_name` 列。
- **`CreatedAt` 和 `UpdatedAt` 字段：**
	- `CreatedAt` 会在记录被创建时自动填充当前时间。
	- `UpdatedAt` 会在记录被更新时自动填充当前时间。
	- 当你用 GORM 创建一条记录时，`CreatedAt` 和 `UpdatedAt` 会被自动设置为当前时间。
	- 当你更新该记录时，`UpdatedAt` 会被自动更新为当前时间，而 `CreatedAt` 保持不变。
	- 通常使用 `time.Time` 类型作为 `CreatedAt` 和 `UpdatedAt` 的字段，你也可以选择使用秒、毫秒或纳秒级的 Unix 时间戳，只需将字段类型设置为 `int64`，并配合使用 `autoCreateTime` 和 `autoUpdateTime` 标签。
- **`DeletedAt` 字段：**
	- `DeletedAt` 是用于软删除的特殊字段。只要你的结构体中包含 `DeletedAt` 字段（类型必须是 `gorm.DeletedAt`），GORM 会自动将其管理起来，用于标记记录是否被删除，而不是实际删除数据库中的记录。
	- 当你调用 GORM 的 `Delete` 方法时，它会将记录的 `DeletedAt` 字段设为当前时间，而不是直接物理删除记录。
	- **查询包括软删除的记录：**
		- ```go
		  db.Unscoped().Where("name = ?", "John").Find(&users)
		  ```
- **`gorm.Model` 结构体：**
	- GORM 提供了一个预定义的 `gorm.Model` 结构体，它包括了常用的字段。你可以在自己的模型中嵌入 `gorm.Model`，它会自动带入这些字段，这样就不需要每次都手动添加 `ID`、`CreatedAt`、`UpdatedAt`、`DeletedAt` 等字段了。
	- ```go
	  type Model struct {
	    ID        uint           `gorm:"primaryKey"`
	    CreatedAt time.Time
	    UpdatedAt time.Time
	    DeletedAt gorm.DeletedAt `gorm:"index"`
	  }
	  
	  type User struct {
	    gorm.Model
	    Name string
	  }
	  ```
- **字段级权限控制：**
	- 可以通过 GORM 的标签来控制每个字段的访问和修改权限。
	- 具体来说，我们可以设置字段为只读、只写、创建时写入、更新时写入等。
	- **常用的权限标签：**
		- **`<-:create`**：仅允许在创建时写入该字段。
		- **`<-:update`**：仅允许在更新时写入该字段。
		- **`<-`**：允许在创建和更新时都能读写该字段。
		- **`->`**：字段为只读，禁止写入，只允许读取。
		- **`<-:false`**：禁止写入（该字段不能写入，无论是在创建还是更新时）。
		- **`-:migration`**：忽略该字段的迁移，不会在数据库中创建该字段。
- **字段标签：**
	- 字段标签（Field Tags）是 Go 语言中结构体字段的元数据，用于给字段附加额外的描述信息。
	- `column`：用于指定结构体字段映射到数据库中的列名。
	- `type`：用于指定字段的数据库数据类型。GORM 会根据字段的类型自动推测数据库的列类型，但你也可以通过 `type` 标签手动指定。
	- `primaryKey`：指定该字段是主键，如果你的主键不叫 `ID`，就需要这个标签。
	- `unique`：指定该字段在数据库中必须是唯一的。使用该标签后，GORM 会在创建数据库表时为该字段创建唯一索引，不允许列中出现重复的值。
	- `index`：你可以使用 `index` 标签为某个字段创建普通索引，大幅提升查询速度。
	- `default`：
		- 当创建一条新记录，如果没有给某个字段赋值，GORM 会使用这里指定的默认值。
		- 当创建记录时，如果该字段是其类型的零值（如 `int` 的 `0`，`string` 的 `""`），GORM 会使用这个默认值。
		- 如果你想插入零值本身（比如 `Age: 0`），GORM 还是会用默认值 `18` 覆盖。为了避免这种情况，你可以使用指针类型 `*int`。
	- `not null`：强制要求这个字段在存入数据库时必须有值，不能为空。
	- `size`：指定字段的最大长度。常用于字符串类型字段，告诉数据库该字段最多可以存储多少字符。
	- `-`：完全忽略该字段。既不会在数据库表中创建这个字段，也不能对它进行任何的读写操作。
- **嵌套结构体：**
	- 你可以在一个结构体中直接嵌入另一个结构体，把它们的字段作为自身的字段。
	- **示例：**
		- ```go
		  type Author struct {
		    Name  string
		    Email string
		  }
		  
		  type Blog struct {
		    Author
		    ID      int
		    Upvotes int32
		  }
		  ```
		- 在 `Blog` 中，`Author` 结构体的字段会直接嵌套到 `Blog` 中，最终在数据库中会生成 `Name` 和 `Email` 两个列。
		- 如果你想给嵌套字段加上前缀，可以使用 `embeddedPrefix` 标签：
			- ```go
			  type Blog struct {
			    Author `gorm:"embedded;embeddedPrefix:author_"`
			  }
			  ```
			- 这样，数据库表中就会出现 `author_name` 和 `author_email` 作为列名。