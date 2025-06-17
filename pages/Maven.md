- `<packaging>`
	- 指定项目构建后生成的产物类型。
	- `jar`
		- 生成 `.jar` 包，用于标准 Java 应用。
	- `war`
		- 生成 `.war` 包，用于部署到 Web 应用服务器。
	- `pom`
		- 用于标识父项目，表示这是一个聚合模块，即作为容器存在，主要负责组织多个子模块，统一管理它们的配置。
		- 该项目本身没有 `src/main/java` 目录，即不包含业务代码，因此不会参与编译，也不会被打包为 `.jar` 或 `.war` 文件。
- `<modules>`
	- 其中填写的是子模块（子项目）的文件夹名称，而不是 `artifactId` 或其它名。
	- **示例：**
		- 假设你的项目目录结构如下：
		- ```text
		  spring-cloud-demo/       <-- 父项目目录
		  ├── pom.xml              <-- 父项目 POM 文件
		  ├── service-user/        <-- 子模块 1
		  │   └── pom.xml
		  ├── service-order/       <-- 子模块 2
		  │   └── pom.xml
		  └── common-utils/        <-- 子模块 3
		      └── pom.xml
		  ```
		- 那么父项目的 `pom.xml` 中应写为：
		- ```xml
		  <modules>
		      <module>service-user</module>
		      <module>service-order</module>
		      <module>common-utils</module>
		  </modules>
		  ```
- `<dependencyManagement>`
	- 用于在父项目中统一管理子模块所需依赖的版本。
	- 在多模块项目中，多个子模块可能会用到相同的依赖，如果每个子模块都单独声明版本，不仅冗余，还容易导致版本不一致的问题。
	- `dependencyManagement` 的作用仅限于声明依赖的版本号，并不实际引入依赖。如果子模块未显式添加该依赖，Maven 不会自动补上。
	- **示例一（手动指定单个依赖版本）：**
		- 父项目通过 `dependencyManagement` 统一指定依赖版本：
		- ```xml
		  <dependencyManagement>
		      <dependencies>
		          <dependency>
		              <groupId>com.fasterxml.jackson.core</groupId>
		              <artifactId>jackson-databind</artifactId>
		              <version>2.16.0</version>
		          </dependency>
		      </dependencies>
		  </dependencyManagement>
		  ```
		- 子模块中可以直接引入该依赖，而无需指定版本：
		- ```xml
		  <dependency>
		      <groupId>com.fasterxml.jackson.core</groupId>
		      <artifactId>jackson-databind</artifactId>
		  </dependency>
		  ```
		- Maven 会根据父项目中的配置，自动为该依赖补全版本号 `2.16.0`。
	- **示例二（版本清单）：**
		- 用于统一管理大量依赖的版本，尤其适用于像 Spring Cloud 这样包含多个子组件（如 Feign、Gateway、Nacos、Config、Bus 等）的套件，这些组件之间的版本必须严格匹配。
		- ```xml
		  <properties>
		      <spring-cloud.version>2025.0.0</spring-cloud.version>
		  </properties>
		  <dependencyManagement>
		      <dependencies>
		          <dependency>
		              <groupId>org.springframework.cloud</groupId>
		              <artifactId>spring-cloud-dependencies</artifactId>
		              <version>${spring-cloud.version}</version>
		              <type>pom</type>
		              <scope>import</scope>
		          </dependency>
		      </dependencies>
		  </dependencyManagement>
		  ```
		- 这段配置的作用是告诉 Maven：“我要导入一个 Spring Cloud 的 BOM 文件，它已经为所有组件（如 Feign、Gateway、Config 等）定义好了版本。”
		- 这样一来，在项目中使用这些组件时就无需手动指定版本，Maven 会自动从 BOM 中获取对应的版本号。
		- `<type>pom</type>`
			- 表示这个依赖不是普通的 JAR 包，而是一个 POM 类型的文件。
			- 它不会参与编译、运行，而是只包含一堆“依赖版本声明”。
			- 就像一本说明书，只告诉你各种零件用哪个版本，并不包含零件本身。
		- `<scope>import</scope>`
			- 表示导入这个 `pom` 文件的内容，作为依赖版本说明书（BOM）来用。
			- 只能出现在 `<dependencyManagement>` 里，用来统一版本。
			- 如果没有它，Maven 不会“导入”这个 POM 里的依赖版本声明。
		- `<type>` 和 `<scope>` 必须成对使用的，专用于导入 BOM 文件。
- `<parent>`
	- `parent` 写在子模块的 `pom.xml` 中，用于声明当前项目继承某个父项目的构建配置，从而复用父项目中定义的依赖版本、插件配置、属性等，避免重复编写，提升可维护性。
	- 即使继承了父项目，子模块仍需明确指定自己的 `artifactId`。
	- 可继承的内容包括：
		- `<dependencyManagement>`
		- `<build>`
		- `<properties>`
		- ...
- `<properties>`
	- `<properties>` 就像是“变量声明区”，你可以在这里统一定义一些值，在其它地方通过 `${变量名}` 重复使用。
	- 它能有效避免重复写相同的内容，例如 Java 版本号。如果你在多个地方使用了 Java 17，只需在 `<properties>` 中定义一次 `${java.version}`，以后修改只需改这一处。
	- 在父项目中定义的变量，子模块也能直接使用，无需重复配置。
	- **示例：**
		- 在 `<properties>` 标签里这样写：
		- ```xml
		  <properties>
		      <java.version>17</java.version>
		      <spring-boot.version>3.1.0</spring-boot.version>
		  </properties>
		  ```
		- 这样你就为这些值起了名字，之后在其他地方只需用 `${java.version}` 来代表 17。
-