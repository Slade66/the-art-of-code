- **作用：**让一个 Goroutine（通常是主程）等待一组 Goroutine 执行完毕。
- **核心原理：计数器**
	- `WaitGroup` 内部维护了一个计数器。
	- **启动任务时**：计数器 `+1`。
	- **任务完成时**：计数器 `-1`。
	- **等待时**：如果计数器大于 0，就一直阻塞（等待）；直到计数器变为 0，才继续执行。
- `func (wg *WaitGroup) Add(delta int)`：修改计数器。
  collapsed:: true
	- 通常在启动 Goroutine 之前调用 `Add(1)`。
	- 也可以一次性添加多个，如 `Add(5)`。
	- **Add 必须在 go 关键字之前调用**
		- 如果在 go 协程内部 Add，可能主程跑得太快，已经运行到 Wait 了，还没看到计数器，就直接结束了，协程还没来得及 Add。
- `func (wg *WaitGroup) Done()`：计数器减 1。
  collapsed:: true
	- 通常在 Goroutine 内部，任务执行结束时调用。
	- 使用 `defer wg.Done()` 确保即使函数 panic 也能正确减数。
	- 如果你调用 `Done()` 的次数超过了 `Add()` 的次数，把计数器被减为负数，会导致 panic。
- `func (wg *WaitGroup) Wait()`：阻塞代码运行。调用此方法的地方会暂停执行，直到计数器归零。
- **WaitGroup vs Channel**
	- **使用 `WaitGroup`**：它是纯粹的控制流同步。当你不需要从 Goroutine 获取结果，或者结果已经被写入了共享内存/切片，你只关心“所有事情是否都做完了”。
	- **使用 `Channel`**：当你需要收集数据，或者需要在 Goroutine 之间进行通信时。
- **代码示例：**
	- ```go
	  package main
	  
	  import (
	  	"fmt"
	  	"sync"
	  	"time"
	  )
	  
	  func worker(id int, wg *sync.WaitGroup) {
	  	defer wg.Done()
	  	fmt.Printf("%d start...\n", id)
	  	time.Sleep(time.Second)
	  	fmt.Printf("%d done...\n", id)
	  }
	  
	  func main() {
	  
	  	var wg sync.WaitGroup
	  	fmt.Println("run...")
	  
	  	for i := 0; i < 5; i++ {
	  		wg.Add(1)
	  		go worker(i, &wg)
	  	}
	  
	  	fmt.Println("wait...")
	  	wg.Wait()
	  
	  }
	  
	  ```
- **注意：**
	- **函数传参时“值拷贝”了 WaitGroup**
		- Go 语言中结构体是值传递的。如果你把 `wg` 作为参数传递给函数，它会被复制一份。子函数里的 `Done()` 减的是副本的计数器，主程里的 `wg` 计数器永远不会变，导致死锁（Deadlock）。
		- ```go
		  func worker(wg sync.WaitGroup) { // ❌ 错误：这里发生了值拷贝
		      defer wg.Done() 
		  }
		  
		  func worker(wg *sync.WaitGroup) { // ✅ 正确：传递指针
		      defer wg.Done()
		  }
		  ```
-