---
创建日期: 2025-04-03
---

#### 用 jps + jstack

`jps` 是随 JDK 一同安装的命令行工具，全称为 Java Virtual Machine Process Status Tool，主要用于列出当前系统中所有正在运行的 Java 进程（即 JVM 实例）及其对应的进程 ID（PID）。

直接运行 `jps` 命令会列出当前系统中所有 Java 进程的类名；加上 `-l` 参数，则会显示完整的类名（包括包名）。

`jstack` 是 JDK 提供的一个命令行工具，用于查看 Java 进程的线程堆栈信息（Thread Dump）。

先使用 `jps` 命令查找当前正在运行的 Java 程序的进程 ID（PID），再执行 `jstack <pid>` 获取线程堆栈信息；如果输出中包含 “Found 1 deadlock.” 字样，就说明程序中存在死锁。

#### 用 jconsole

`jconsole` 是 JDK 提供的一个基于图形界面的监控工具，全称是 Java Monitoring and Management Console，用于实时监控 Java 应用程序的运行状态。

执行 `jconsole` 命令启动图形化监控工具，连接到目标 Java 进程后，切换到“线程”标签页，点击“检测死锁”按钮即可查看是否存在死锁。
