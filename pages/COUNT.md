- `COUNT(*)`
  title:: COUNT
	- 统计表中的总行数，包括有 `NULL` 值的行。
- `COUNT(column_name)`
	- 统计指定列中非 `NULL` 值的行的数量。它会忽略值为 `NULL` 的行。
- `COUNT(DISTINCT column_name)`
	- 统计指定列中唯一且非 `NULL` 值的行的数量。
-