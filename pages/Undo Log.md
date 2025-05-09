有什么用？
heading:: true
	- InnoDB 的 Undo Log 是实现事务原子性（ACID 中的 A）、支持事务回滚以及多版本并发控制功能的基础。
	- Undo Log（回滚日志）用于记录数据修改前的旧值。每条记录对应一次具体的写操作，包含还原该操作所需的信息。当事务对某行执行修改时，InnoDB 会将修改前的数据写入 Undo Log。若同一行被多次修改，就会形成一条由 Undo Log 构成的版本链，记录之间通过指针相连，使 InnoDB 能够按需回溯到任意历史版本。
	- **原子性（Atomicity）：**Undo Log 保证了事务的原子性，即事务要么全部执行成功，要么在失败或显式回滚时撤销所有已执行的操作，使数据库恢复到事务开始前的状态，仿佛该事务从未发生。
	- **事务回滚：**当事务需要回滚时，InnoDB 会根据 Undo Log 中保存的修改前数据，将已改动的内容恢复为原始状态，确保数据库回到事务开始前的样子。对于事务执行的每一次修改，InnoDB 都会读取对应的 Undo Log 记录并执行其中的反向操作：`UPDATE` 操作会用旧值覆盖当前行，`INSERT` 会删除新插入的行，`DELETE` 则会撤销删除标记，使数据重新可见。
	- **多版本并发控制（MVCC）：**MVCC 允许读操作（如 `SELECT`）与写操作（如 `INSERT`、`UPDATE`、`DELETE`）并发执行而无需互相阻塞，显著提升了数据库的并发性能。InnoDB 会在 Undo Log 中为每条记录保留修改前的旧版本。当事务读取数据时，会根据自身的启动时间（或语句启动时间，视隔离级别而定）生成一个“读视图”（Read View），并结合记录中的 `DB_TRX_ID` 和 `DB_ROLL_PTR` 字段持续回溯版本链，确定应访问哪个数据版本。整个过程无需加锁，从而实现了非阻塞的一致性读。
- Undo Log 的持久化机制
  heading:: true
	- 很多人误以为 Undo Log 在写入后会立即单独刷盘以确保数据安全，实际上并非如此。Undo Log 被存放在 Undo 页中，这些页与普通数据页一样，存在于内存中的 Buffer Pool 中。
	- 对 Undo Log 的修改并不会直接写入磁盘，而是通过 Redo Log 间接实现持久化：每次对 Undo 页的更改，都会同时记录到 Redo Log 中。只要这些 Redo Log 在事务提交时或定时刷盘时写入磁盘，InnoDB 就能够在崩溃恢复时重做 Undo 页的修改，从而间接保证了 Undo Log 的持久性。
	- 示例
	  heading:: true
		- 当你执行了一条更新语句：`UPDATE users SET age = 25 WHERE id = 1;`
		- InnoDB 会按以下流程处理：
			- 读取数据：从 Buffer Pool（内存）中加载 `id = 1` 的记录，假设原始值为 `age = 20`。
			  logseq.order-list-type:: number
			- 生成 Undo Log：
			  logseq.order-list-type:: number
				- 内容：记录修改前的旧值（`age = 20`），写入 Undo 页；
				- 目的：
					- 以后如果事务回滚，可以撤销这次修改；
					- 支持其它事务通过 MVCC 读取旧版本数据。
			- 生成 Redo Log
			  logseq.order-list-type:: number
				- 内容：
					- 对数据页的修改（`age` 从 20 改为 25）；
					- 对 Undo 页的修改（添加一条 Undo Log 记录）；
				- 作用：在数据库崩溃后重做这些修改，确保数据和 Undo Log 都能恢复。
			- 事务提交前刷盘 Redo Log
			  logseq.order-list-type:: number
				- 虽然此时 Undo 页和数据页还未刷盘，但只要 Redo Log 已经持久化，就能在崩溃恢复时重做对 Undo 页的修改。
-