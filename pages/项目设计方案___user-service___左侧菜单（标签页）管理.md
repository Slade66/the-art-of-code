## 概述
	- 本质上是基于角色的访问控制（RBAC）在前端界面上的应用。
- ## 目标
	- 左侧菜单（标签页）按“当前登录用户”的权限显示/隐藏。
- ## 数据模型设计
	- ### 用户表 (User)
		- | **字段名** | **英文名** | **数据类型** | **描述** |
		  | ---- | ---- | ---- |
		  | **用户ID** | `user_id` | INT / BIGINT | **主键**。唯一标识一个用户。 |
		  | **用户名** | `username` | VARCHAR | 用户的登录名。 |
		  | **...** | ... | ... | 其他登录相关信息，如密码哈希等（略）。 |
		-
	- ### 菜单表 (Menu)
		- | **字段名** | **英文名** | **数据类型** | **描述** |
		  | ---- | ---- | ---- |
		  | **菜单ID** | `menu_id` | INT / BIGINT | **主键**。唯一标识一个菜单项。 |
		  | **菜单名称** | `menu_name` | VARCHAR | 左侧导航栏显示的名称（如：`仓库列表`）。 |
		  | **路由路径** | `path` | VARCHAR | 前端跳转的 URL 路径（如：`/warehouse/list`）。 |
		  | **上级菜单ID** | `parent_id` | INT / BIGINT | 用于构建**嵌套菜单**的父级 ID。**0 或 NULL** 表示一级菜单。 |
		  | **排序值** | `sort_order` | INT | 菜单在左侧显示的顺序。 |
	- ### 用户-菜单关联表 (User_Menu_Mapping)
		- | **字段名** | **英文名** | **数据类型** | **描述** |
		  | ---- | ---- | ---- |
		  | **用户ID** | `user_id` | INT / BIGINT | **外键**，关联 `User` 表。 |
		  | **菜单ID** | `menu_id` | INT / BIGINT | **外键**，关联 `Menu` 表。 |
		- 这张表的**联合主键**是 `(user_id, menu_id)`，确保一个用户不会重复拥有同一个菜单权限。
- ## 后端接口设计
	- 后端查询逻辑：当用户登录时，通过 `user_id` 查询用户-菜单关联表，得到他所有有权限的 `menu_id` 列表。然后拿这些 `menu_id` 去 `Menu` 表里查询详情，并构建菜单树返回给前端。
	- 后端的主要目标是：根据当前登录用户的ID，从数据库中查询出他有权限访问的菜单列表，并以易于前端渲染的格式返回。
	- #### 根据 user_uid 获取菜单列表。
		- 参数：user_uid
		- 路径：/menu/get-menus
		- 返回值：
			- ```
			  [
			    {
			      "menu_id": 100,
			      "menu_name": "系统管理",
			      "path": "/system",
			      "sort_order": 1,
			      "children": [
			        {
			          "menu_id": 101,
			          "menu_name": "用户管理",
			          "path": "/system/user",
			          "sort_order": 1
			        },
			        {
			          "menu_id": 102,
			          "menu_name": "仓库列表",
			          "path": "/warehouse/list",
			          "sort_order": 2
			        }
			      ]
			    },
			    {
			      "menu_id": 200,
			      "menu_name": "个人中心",
			      "path": "/profile",
			      "sort_order": 2,
			      "children": []
			    }
			  ]
			  ```
		- 伪代码逻辑：
			- ```
			  // 1. 认证与用户识别
			  function handle_get_user_menus(request):
			      // 从请求头部或会话中解析出用户ID
			      user_id = extract_user_id_from_token(request) 
			      
			      if not user_id:
			          return 401 Unauthorized Error
			  
			      // 2. 数据库查询：获取用户有权限的菜单
			      SQL_QUERY = """
			          SELECT m.menu_id, m.menu_name, m.path, m.parent_id, m.sort_order
			          FROM Menu m
			          JOIN User_Menu_Mapping um ON m.menu_id = um.menu_id
			          WHERE um.user_id = :user_id
			          ORDER BY m.sort_order ASC;
			      """
			      
			      // 执行查询，得到一个扁平的菜单列表
			      flat_menu_list = execute_query(SQL_QUERY, {user_id: user_id})
			  
			      // 3. 数据处理：构建树形结构
			      // 菜单项通常是嵌套的（一级菜单、二级菜单）。前端渲染树形结构更方便。
			      menu_tree = build_tree_from_flat_list(flat_menu_list)
			  
			      // 4. 返回结果
			      return 200 OK, menu_tree
			  ```
	- #### 创建菜单 (Create)
	  collapsed:: true
		- **API:** `POST /api/menus`
		- 请求体：包含 `MenuName`, `Path`, `ParentID`, `SortOrder` 的 JSON 对象。
		- 逻辑：
			- ```
			  // 1. 业务逻辑校验：例如 Path 是否已存在、如果有 ParentID，它是否有效
			  
			  ```
	- #### 更新菜单 (Update)
	  collapsed:: true
		- **请求体:** 包含需要更新的字段（如 `MenuName`, `Path` 等）。
	- **删除菜单 (Delete)**
	- 获取全部菜单
	  collapsed:: true
		- **用途:** 主要用于管理后台展示完整的菜单项。
	- 获取单个菜单。
-