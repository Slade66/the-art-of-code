- **作用：**
	- 用于在读取器（Reader）和写入器（Writer）之间高效传输数据。
	- 它从 `src` 读取数据并写入到 `dst`，直到 `src` 返回 `EOF` 或发生错误。
- **方法签名：**
	- ```go
	  func Copy(dst Writer, src Reader) (written int64, err error)
	  ```
- **参数：**
	- `dst Writer`：写入数据的目的地，任何实现了 `io.Writer` 接口的类型都可作为目的地。
	- `src Reader`：读取数据的源头，任何实现了 `io.Reader` 接口的类型都可作为数据源。
- **返回值：**
	- `written int64`：成功拷贝的字节数。
	- `err error`：拷贝过程中，若出现磁盘已满、网络断开或权限不足等意外，`io.Copy` 会返回非 `nil` 的 `err`；若无异常，则返回 `nil`，代表拷贝成功。
- **核心工作流程：**
	- 检查 `src` 是否实现了 `WriterTo` 接口，如果实现，则直接调用 `src.WriteTo(dst)` 进行数据复制。
	  logseq.order-list-type:: number
	- 检查 `dst` 是否实现了 `ReaderFrom` 接口，如果实现，则直接调用 `dst.ReadFrom(src)` 进行数据复制。
	  logseq.order-list-type:: number
	- 创建一个 32 KB 的缓冲区。
	  logseq.order-list-type:: number
	- 在循环中不断从 `src` 读取数据块填充到缓冲区。
	  logseq.order-list-type:: number
	- 然后，将缓冲区中的数据写入 `dst`。
	  logseq.order-list-type:: number
	- 这个循环会持续执行，直到 `src.Read` 返回 `io.EOF`（文件结束）信号，表示数据源已读取完毕，或在读写过程中发生其它错误。
	  logseq.order-list-type:: number
- **注意：**
	- `io.Copy` 会首先检查 `src` 和 `dst` 是否能自行处理数据复制。如果可以，`io.Copy` 会将该任务交由它们完成，这样便减少了创建缓冲区等不必要的步骤。