有什么用？
heading:: true
	- Redo Log 就像一份“修改备忘录”，记录事务对数据所做的更改操作。
	- InnoDB 的 Redo Log（重做日志）是实现事务持久性（ACID 模型中的 D）以及确保数据库在意外崩溃后能够恢复到一致状态的关键机制。
	- **持久性（Durability）：**Redo Log 采用预写日志策略，确保一旦事务提交，其修改就被永久保存。即使这些更改尚未同步到磁盘上的数据文件，也不会丢失。Redo Log 记录的是对数据页的物理修改。
	- **崩溃恢复（Crash Recovery）：**当数据库意外宕机并重启时，InnoDB 会自动检查并重放 Redo Log 中记录的已提交但尚未写入数据文件的操作，从而将数据库恢复到崩溃前的最后一致状态，确保已提交事务的修改不会丢失。
- 崩溃恢复
  heading:: true
	- WAL（Write-Ahead Logging，预写日志）
	  heading:: true
		- 当一条记录需要更新时，InnoDB 引擎会首先更新内存中的数据页，并将其标记为脏页。同时，这次对数据页的修改也会以 redo log 的形式记录到内存中的 Redo Log Buffer 中。此时，更新操作即视为完成。随后，这些日志会被刷新到磁盘上的 Redo Log 文件中。之后，InnoDB 会在适当的时机，由后台线程将 Buffer Pool 中的脏页写入磁盘。
		- MySQL 的写操作采用“先写日志，再写数据”的策略：并不是立即将数据写入磁盘，而是先持久化 redo log，再异步地将数据页写入磁盘。这样即便在脏页尚未写入磁盘时发生系统崩溃，由于 redo log 已经持久化，MySQL 在重启后仍可根据 redo log 恢复数据，将数据库还原到崩溃前的最新状态，从而确保已提交的数据不会丢失，实现崩溃恢复能力。
	- 例子 1
	  heading:: true
		- 假设你在业务中执行了如下语句：`UPDATE users SET age = 25 WHERE id = 1;`
		- 此操作会经历以下几个步骤：
			- 内存更新：InnoDB 将 `id = 1` 的用户的 `age` 字段修改为 25，更新后的数据页保存在内存（Buffer Pool）中，并被标记为“脏页”。
			- 写入 Redo Log：该修改被记录到内存中的 Redo Log Buffer，随后刷新到磁盘上的 redo log 文件中。
			- 事务提交：数据库确认事务提交成功。注意，此时数据页尚未写入磁盘，只有 redo log 已持久化。
		- 突然，MySQL 服务器崩溃了，`users` 表对应的数据页还没来得及写入磁盘。
		- MySQL 重启后，InnoDB 会执行崩溃恢复流程：
			- 首先定位上一次的检查点 LSN。检查点之前的数据已经持久化，因此只需从该位置开始恢复。
			- 从该检查点开始，顺序读取 redo log 中的记录。
			- 发现你刚才执行的更新操作：
				- 若 `id = 1` 的数据页仍在 Buffer Pool 中，直接应用该修改；
				- 否则，先从磁盘加载该页，再根据 redo log 进行恢复。
			- 应用完成后，`age` 字段恢复为 25，数据恢复为一致状态。
	- 例子 2
	  heading:: true
		- 在游戏中，每完成一个任务，系统都会记录一个带编号的保存点，例如保存点 1000、2000、3000。这些编号就类似于数据库中的 LSN（Log Sequence Number）。
		- 假设你打完第一关，系统记录了保存点 1000；接着完成第二关和第三关，分别生成保存点 2000 和 3000。
		- 这些保存信息不会立即写入硬盘，而是先被记录在日志中，这就像数据库中的 Redo Log。
		- 如果这时突然断电，硬盘上还停留在保存点 1000，表面上看第二关和第三关的进度似乎丢失了。
		- 重启后，系统先加载硬盘上的旧存档（保存点 1000），然后检查日志。
		- 日志中发现你实际上已经达到了保存点 3000，于是系统根据日志内容“重放”中间的操作，最终将游戏恢复到崩溃前的最新状态。
- Redo Log 和 Undo Log 的区别
  heading:: true
	- 它们都是 InnoDB 存储引擎层的日志。
	- Redo Log 记录数据修改后的新值，在数据库故障重启时通过重做已提交的操作，保障事务持久性；Undo Log 保存修改前的旧值，用于事务回滚时撤销操作以确保原子性，同时为多版本并发控制（MVCC）提供支持。
- Redo Log 的写盘机制
  heading:: true
	- Redo Log 并非直接写入磁盘，而是通过 Redo Log Buffer 缓冲机制减少磁盘 I/O。每当生成一条 Redo Log 时，会先暂存于 Redo Log Buffer，后续再持久化到磁盘。
	- 触发刷盘的时机：
	  heading:: true
		- **MySQL 正常关闭：**关闭时会将所有缓冲日志强制落盘。
		- **缓冲空间阈值触发：**当 Redo Log Buffer 已使用空间超过其容量的一半时，自动触发落盘。
		- **定时刷盘机制：**InnoDB 后台线程每隔 1 秒，会将 Redo Log Buffer 里的记录持久化到磁盘。
		- **事务提交：**每次事务提交时都会将 Redo Log Buffer 里的记录持久化到磁盘。
	- 事务提交的刷盘策略
	  heading:: true
		- 事务提交时 Redo Log 的刷盘行为由 `innodb_flush_log_at_trx_commit` 参数控制。
		- `= 0`
			- **行为：**
				- 事务提交时，Redo Log 仅保留在 Buffer 中，不主动触发磁盘写入。
				- 后台线程每隔 1 秒执行两步操作：
					- 先通过 `write()` 将日志写入操作系统的 Page Cache（文件缓存）；
					- 再通过 `fsync()` 将缓存数据强制持久化到磁盘。
			- **风险与性能：**
				- 若 MySQL 进程崩溃，未到刷盘时间的近 1 秒内事务数据会丢失（因日志仍在 Buffer 中）。
				- 性能优势明显，避免了事务提交时的磁盘 I/O 等待，但数据持久性最低。
		- `= 1`（默认）
			- **行为：**
				- 每次事务提交时，InnoDB 直接将 Buffer 中的日志写入 Page Cache，并立即调用 `fsync()` 强制数据落盘。
			- **风险与性能：**
				- 确保事务提交后日志永久保存，即使操作系统崩溃或断电也不丢失，完全满足 ACID 持久性要求。
				- 缺点是每次提交需等待磁盘 I/O 完成，在高并发写入场景可能成为性能瓶颈。
		- `= 2`
			- **行为：**
				- 事务提交时，日志仅写入操作系统的 Page Cache（不调用 `fsync()`），由系统每隔 1 秒自动执行 `fsync()` 落盘。
			- **风险与性能：**
				- 安全性介于 0 和 1 之间：MySQL 进程崩溃不会丢失数据（日志在 Page Cache 中），但操作系统崩溃或断电可能导致近 1 秒内的事务数据丢失。
				- 性能优于参数 1，事务提交无需等待磁盘 I/O，延迟显著降低，但依赖系统定时刷盘机制。
- Redo Log 的循环写入与空间管理机制
  heading:: true
	- **Redo Log 文件组的循环写入特性：**
		- Redo Log 物理上存储于一组文件中，InnoDB 采用循环写入方式：当写入位置到达最后一个文件末尾时，会回到第一个文件开始覆盖已确认不再需要的旧日志记录。这种环形结构通过两个关键指针管理：
			- **write pos：**标识当前日志记录的写入位置；
			- **checkpoint：**标识可擦除旧日志的起始位置，两者之间的未擦除区域为有效日志空间。
		- ![](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/how_update/checkpoint.png){:height 506, :width 748}
	- **日志擦除的触发条件：**
		- Redo Log 的设计初衷是防止 Buffer Pool 中脏页丢失。当脏页刷新到磁盘后，其对应的 Redo Log 记录便不再需要，此时旧日志可被擦除以腾出空间。正常运行时，日志以 write pos 为起点顺序写入，当 write pos 追上 checkpoint（即日志空间用尽），MySQL 会阻塞新的更新操作，强制将 Buffer Pool 中的脏页刷新到磁盘，并标记可擦除的旧日志区域。随后 checkpoint 指针后移（顺时针移动），释放的空间用于接收新日志，系统恢复正常写入。
	- **Redo Log 容量对性能的影响：**
		- 较大的 Redo Log 容量可延长检查点间隔，允许数据库在写入高峰期积累更多脏页，推迟可能影响性能的强制检查点刷新，从而平滑磁盘 I/O 负载。反之，较小的容量会导致检查点更频繁触发，可能引发写入性能瓶颈，但故障恢复时间相对更短。