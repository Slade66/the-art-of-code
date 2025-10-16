有什么用？
heading:: true
	- Binlog 用于记录所有对 MySQL 数据库进行更改的语句（不包括如 `SELECT` 这类不修改数据的操作），用于实现数据恢复和主从复制。
	- **主从复制：**主服务器会将所有数据更改操作记录到 Binlog 中，并将这些日志事件传送给从服务器。从服务器通过读取并重放这些日志事件，实现与主服务器的数据同步。
	- **数据恢复：**当数据库发生故障或误操作时，可先恢复最近一次的全量备份，然后基于该备份起点，按时间顺序回放 Binlog 中的变更操作，从而将数据恢复到指定的时间点或状态。
- Binlog 的写入时机
  heading:: true
	- 两阶段提交
	  heading:: true
		- 在 MySQL 中，事务的提交采用“两阶段提交”机制，以确保 Redo Log 与 Binlog 之间的数据一致性，防止主从复制时出现数据不一致问题。
		- 两阶段提交的流程如下：
			- 执行 SQL，修改 Buffer Pool 中的内存数据页；
			  logseq.order-list-type:: number
			- 写入 Redo Log，并将其标记为 `prepare` 状态；
			  logseq.order-list-type:: number
			- 写入 Binlog；
			  logseq.order-list-type:: number
			- Redo Log 写入 `commit` 标记。
			  logseq.order-list-type:: number
		- 只有在 Binlog 成功写入并持久化到磁盘后，MySQL 才会通知 InnoDB 存储引擎将事务正式标记为“已提交”。换句话说，事务只有在 Redo Log 处于“准备”状态且 Binlog 已成功持久化后，才算真正完成提交。
		- MySQL 的主从复制依赖 Binlog 文件来重放主库的操作。由于 Binlog 只记录已提交的事务，从库可以安全地重放这些操作，从而确保与主库状态一致。两阶段提交正是实现这种复制可靠性的关键机制。
	- 为什么需要两阶段提交？
	  heading:: true
		- 如果缺乏两阶段提交的协调机制，可能导致以下数据不一致问题：
			- **Redo Log 已提交，但 Binlog 未写入**：事务修改已持久化到数据页，但 Binlog 中没有该事务，从库无法复制这个事务，导致主从不一致，或者基于 Binlog 的恢复时丢失该事务。
			- **Binlog 已写入，但 Redo Log 未提交**：Binlog 记录了事务，但数据库实际并未完成提交。崩溃恢复后，从库会执行这个事务，而主库并未真正应用，导致主从不一致。
	- 崩溃恢复下的两种场景
	  heading:: true
		- **崩溃发生在 Redo Log 的 prepare 持久化后，但 Binlog 写入之前**：
			- InnoDB 恢复时发现事务处于“准备”状态，会去 Binlog 查询是否有对应记录。
			- 如果 Binlog 中不存在该事务，说明事务未提交，将通过 Undo Log 回滚。
		- **崩溃发生在 Binlog 写入后，但 Redo Log commit 尚未完成**：
			- 恢复时发现事务处于“准备”状态，并在 Binlog 中存在记录。
			- InnoDB 会将其视为已提交事务，通过 Redo Log 进行前滚，确保所有变更生效。
- Binlog 记录的内容是什么？
  heading:: true
	- Binlog 是一种逻辑日志，用于记录对数据的操作“意图”，而不是记录最终的数据页内容。根据格式不同，其记录方式也有所区别：
		- **STATEMENT 格式**：记录执行的 SQL 语句本身。这种方式效率较高，但由于某些语句具有不确定性（如使用 `UUID()`、`NOW()` 等），可能导致主从数据不一致。
		- **ROW 格式**：不记录 SQL 语句，而是精确记录每一行数据的变更内容，包括修改前和修改后的数据，能更可靠地保证主从一致性。
		- **MIXED 格式**：默认采用 STATEMENT 格式。当遇到对 STATEMENT 格式不安全的语句时，MySQL 会自动切换为 ROW 格式来记录对应事件，以兼顾性能与安全性。
- Redo Log 和 Binary Log 的区别
  heading:: true
	- **作用层级不同**：Redo Log 是 InnoDB 存储引擎专用的日志，用于保障本地事务的持久性；而 Binlog 属于 MySQL Server 层，适用于所有存储引擎，主要用于主从复制和数据恢复。
	- **功能侧重点不同**：
		- Redo Log 主要用于在系统崩溃后恢复到“最后一致性状态”，确保本地数据不会丢失；Binlog 则用于记录所有数据更改的逻辑操作，支持主从复制以及按时间点的数据恢复。
		- 如果不小心将整个数据库的数据删除，无法通过 Redo Log 恢复，只能依赖 Binlog。这是因为 Redo Log 采用循环写入方式，仅用于恢复尚未刷入磁盘的变更数据；一旦数据持久化，相应日志就会被覆盖，无法用于还原历史状态。而 Binlog 记录了所有数据变更操作，按顺序追加写入，不会被覆盖。只要相关操作被记录在 Binlog 中，就可以通过重放实现数据恢复。
	- **记录内容层级不同**：Redo Log 记录的是物理层的页级变更，例如“将磁盘上 page 123 的第 5 行第 7 字节改为 0x11”；Binlog 记录的是逻辑操作，如 SQL 语句或行数据的变更，例如“执行了 `UPDATE users SET name='小泽' WHERE id=1`”。
	- **写入方式不同**：Redo Log 是固定大小、循环写入的日志文件；而 Binlog 是按顺序追加写入的多文件结构，持续增长。