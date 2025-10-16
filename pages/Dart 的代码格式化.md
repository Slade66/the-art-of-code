- Dart 自带代码格式化工具 —— `dart format`。
- 从 Dart 3.7 起，格式化行为有所调整：如果一行能容纳完整代码，格式化时会将其合并为单行，并移除末尾的逗号。
- Flutter 支持通过配置文件 `analysis_options.yaml` 自定义格式化行为，例如保留尾逗号：
	- ```yaml
	  formatter:
	    trailing_commas: preserve
	  ```
	- 启用该配置后，`dart format` 会保留末尾逗号，并更倾向于使用多行格式。
-