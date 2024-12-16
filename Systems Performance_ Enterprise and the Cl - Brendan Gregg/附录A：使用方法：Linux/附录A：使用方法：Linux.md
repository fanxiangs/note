# 附录A

使用方法：Linux

本附录包含源自 USE 方法 \[Gregg 13d\] 的 Linux 清单。这是一种检查系统健康状况并识别常见资源瓶颈和错误的方法，在第 2 章“方法论”第 2.5.9 节“USE 方法”中介绍。后面的章节（第 5、6、7、9、10）在特定上下文中描述了它，并介绍了支持其使用的工具。

性能工具通常会得到增强，并且会开发新的工具，因此您应该将此视为需要更新的起点。还可以开发新的可观察性框架和工具，专门使遵循 USE 方法变得更容易。

## 物质资源

成分

类型

公制

中央处理器

利用率

每个 CPU：`mpstat -P ALL 1`，CPU 消耗列的总和（`%usr`、`%nice`、`%sys`、`%irq`、`%soft`、`%guest`、`%gnice`) 或空闲列的倒数（`%iowait`、`%steal`、`%idle`）； `sar -P ALL`、消耗 CPU 的列 (`%user`、`%nice`、`%system`) 的总和或空闲列的倒数 (`%iowait`、`%steal`、`%idle`）

全系统：`vmstat 1`、`us` + `sy`； `sar -u`、`%user` + `%nice` + `%system`

每个进程：`top`、`%CPU`； `htop`、`CPU%`； `ps -o pcpu`； `pidstat 1`、`%CPU`

每个内核线程：`top/htop`（`K` 进行切换），其中 VIRT == 0（启发式）

中央处理器

饱和

系统范围：`vmstat 1`、`r` > CPU 计数[^1]； `sar -q`、`runq-sz` > CPU 计数； `runqlat`； `runqlen`

每个进程：/proc/PID/schedstat 第二个字段（sched\_info.run\_delay）； getdelays.c，`CPU`[^2]； `perf sched latency`（显示每个时间表的平均和最大延迟）[^3]

中央处理器

错误

`dmesg` 或 rasdaemon 和 `ras-mc-ctl --summary` 中出现机器检查异常 (MCE)； perf(1) 如果处理器特定的错误事件 (PMC) 可用；例如，AMD64的“04Ah Single-bit ECC Errors Recorded by Scrubber”[^4]（也可以归类为内存设备错误）； `ipmtool sel list`； `ipmitool sdr list`

内存容量

利用率

系统范围：`free -m`、`Mem:`（主内存）、`Swap:`（虚拟内存）； `vmstat 1`、`free`（主内存）、`swap`（虚拟内存）； `sar -r`、`%memused`； `slabtop -s c` 用于 kmem 板的使用

每个进程：`top`/`htop`、`RES`（驻留主内存）、`VIRT`（虚拟内存）、`Mem` 用于系统范围摘要

内存容量

饱和

全系统：`vmstat 1`、`si`/`so`（交换）； `sar -B`、`pgscank` + `pgscand`（扫描）； `sar -W`

每个进程：getdelays.c，`SWAP`2； /proc/PID/stat 中的第 10 个字段 (min\_flt) 用于表示轻微故障率，或动态检测[^5]； `dmesg | grep killed`（OOM 杀手）

内存容量

错误

`dmesg` 用于物理故障或 rasdaemon 以及 `ras-mc-ctl --summary` 或 `edac-util`； `dmidecode` 也可能显示物理故障； `ipmtool sel list`； `ipmitool sdr list`；动态检测，例如失败的 malloc() 的 uretprobes (bpftrace)

网络接口

利用率

`ip -s link`，RX/TX tput / 最大带宽； `sar -n DEV`，rx/tx kB/s / 最大带宽； /proc/net/dev，`bytes` RX/TX tput/max

网络接口

饱和

`nstat`、`TcpRetransSegs`； `sar -n EDEV`、`*drop/s`、`*fifo/s`[^6]； /proc/net/dev，RX/TX `drop`；其他 TCP/IP 堆栈队列的动态检测 (bpftrace)

网络接口

错误

`ip -s link`、`errors`； `sar -n EDEV`全部； /proc/net/dev, `errs`, `drop6`;额外的计数器可能位于 /sys/class/net/\*/statistics/\*error\* 下；驱动程序函数返回的动态检测

存储设备输入/输出

利用率

全系统：`iostat -xz 1`、`%util`； `sar -d`、`%util`；每个进程：`iotop`、`biotop`； /proc/PID/sched `se.statistics.iowait_sum`

存储设备输入/输出

饱和

`iostat -xnz 1`、`avgqu-sz` > 1，或高 `await`； `sar -d`相同； perf(1) 块队列长度/延迟的跟踪点； `biolatency`

存储设备输入/输出

错误

/sys/devices/ 。 。 。 /ioerr\_cnt； `smartctl`； `bioerr`； I/O 子系统响应代码的动态/静态检测[^7]

存储容量

利用率

交换：`swapon -s`； `free`； /proc/meminfo `SwapFree`/`SwapTotal`;文件系统：`df -h`

存储容量

饱和

不确定这个是否有意义——一旦满了，ENOSPC（尽管当接近满时，性能可能会下降，具体取决于文件系统空闲块算法）

存储容量

文件系统：错误

ENOSPC 的 `strace`； ENOSPC 动态仪器； /var/log/messages 错误，取决于 FS；应用程序日志错误

存储控制器

利用率

`iostat -sxz 1`，对设备求和并与每卡的已知 IOPS/tput 限制进行比较

存储控制器

饱和

请参阅存储设备饱和度。 。 。

存储控制器

错误

请参阅存储设备错误。 。 。

网络控制器

利用率

从 `ip –s link` （或 `sar` 或 /proc/net/dev）和已知控制器接口的最大 tput 进行推断

网络控制器

饱和

请参阅网络接口、饱和度、。 。 。

网络控制器

错误

请参阅网络接口、错误、。 。 。

CPU互连

利用率

`perf stat`，带 PMC 用于 CPU 互连端口，tput/max

CPU互连

饱和

`perf stat` 带有用于失速周期的 PMC

CPU互连

错误

`perf stat` 与 PMC 合作，尽其所能

内存互连

利用率

`perf stat` 具有用于内存总线的 PMC，tput/max；例如英特尔uncore\_imc/data\_reads/,uncore\_imc/data\_writes/;或 IPC 小于 0.2； PMC 还可能有本地计数器和远程计数器

内存互连

饱和

`perf stat` 带有用于失速周期的 PMC

内存互连

错误

`perf stat` 与 PMC 合作，尽其所能； `dmidecode` 可能有东西

输入/输出互连

利用率

`perf stat` 具有用于 tput/max 的 PMC（如果可用）；通过来自 iostat/ip/ 的已知 tput 进行推断。 。 。

输入/输出互连

饱和

`perf stat` 带有用于失速周期的 PMC

输入/输出互连

错误

`perf stat` 与 PMC 合作，尽其所能

一般说明：`uptime`“平均负载”（或 /proc/loadavg）不包含在 CPU 指标中，因为 Linux 平均负载包括处于不间断 I/O 状态的任务。

perf(1)：是一个强大的可观测性工具包，可以读取 PMC，还可以使用动态和静态检测。它的接口是 perf(1) 命令。请参阅第 13 章，性能。

PMC：性能监控计数器。请参阅第 6 章，CPU 及其与 perf(1) 的用法。

I/O 互连：这包括 CPU 到 I/O 控制器总线、I/O 控制器和设备总线（例如 PCIe）。

动态检测：允许开发自定义指标。请参阅第 4 章“可观察性工具”以及后面章节中的示例。 Linux 的动态跟踪工具包括 perf(1)（第 13 章）、Ftrace（第 14 章）、BCC 和 bpftrace（第 15 章）。

对于任何实施资源控制的环境（例如云计算），请检查每个资源控制的 USE。在充分利用硬件资源之前，可能会遇到这些问题并限制使用。

[^1]`r` 列报告正在等待的线程和正在 CPU 上运行的线程。请参阅第 6 章“CPU”中的 `vmstat(1)` 描述。

[^2]使用延迟核算；请参阅第 4 章，可观测性工具。

[^3]还有 perf(1) 的 sched:sched\_process\_wait 跟踪点；跟踪时要小心开销，因为调度程序事件很频繁。

[^4]最近的 Intel 和 AMD 处理器手册中没有太多与错误相关的事件。

[^5]这可用于通过查看导致小故障的原因来显示消耗内存并导致饱和的原因。这应该在 htop(1) 中作为 MINFLT 提供。

[^6]丢弃的数据包作为饱和和错误指示器包含在内，因为这两种类型的事件都可能导致丢弃数据包。

[^7]这包括来自 I/O 子系统不同层的跟踪功能：块设备、SCSI、SATA、IDE...一些静态探针可用（perf(1) scsi 和块跟踪点事件）；否则，使用动态跟踪。

## 软件资源

成分

类型

公制

内核互斥体

利用率

使用 CONFIG\_LOCK\_STATS=y，/proc/lock\_stat `holdtime-total` / `acquisitions`（另请参阅 `holdtime-min`、`holdtime-max`）[^8]；锁定功能或指令的动态检测（也许）

内核互斥体

饱和

使用 CONFIG\_LOCK\_STATS=y，/proc/lock\_stat `waittime-total` / `contentions`（另请参阅 `waittime-min`、`waittime-max`）；锁定函数的动态检测，例如 `mlock.bt` \[Gregg 19\]；旋转显示并显示分析 `perf record -a -g -F 99` ...

内核互斥体

错误

动态检测（例如，递归互斥输入）；其他错误可能会导致内核锁定/恐慌，请使用 kdump/`crash` 进行调试

用户互斥体

利用率

`valgrind --tool=drd --exclusive-threshold=` ...（举行时间）；锁定解锁功能时间的动态检测[^9]

用户互斥体

饱和

`valgrind --tool=drd` 从持有时间推断争用；动态检测同步函数的等待时间，例如 `pmlock.bt`；分析 (perf(1)) 用户堆栈的旋转

用户互斥体

错误

`valgrind --tool=drd`各种错误； pthread\_mutex\_lock() 的动态检测，适用于 EAGAIN、EINVAL、EPERM、EDEADLK、ENOMEM、EOWNERDEAD、. 。 。

任务能力

利用率

`top`/`htop`、`Tasks`（当前）； `sysctl kernel.threads-max`，/proc/sys/kernel/threads-max（最大值）

任务能力

饱和

线程在内存分配上阻塞；此时页面扫描仪应该正在运行（`sar -B`，`pgscan*`），否则使用动态跟踪进行检查

任务能力

错误

“无法 fork()”错误；用户级线程：pthread\_create() 因 EAGAIN、EINVAL、. 。 。 ;内核：kernel\_thread() 的动态跟踪 ENOMEM

文件描述符

利用率

系统范围：`sar -v`、`file-nr` 与 /proc/sys/fs/file-max；或者只是 /proc/sys/fs/file-nr

每个进程：`echo /proc/PID/fd/* | wc -w` 与 `ulimit -n`

文件描述符

饱和

这可能没有意义

文件描述符

错误

`strace` errno == EMFILE 在返回文件描述符的系统调用上（例如，open(2)、accept(2)、...）； `opensnoop -x`

[^8]内核锁分析过去是通过lockmeter进行的，它有一个名为lockstat的接口。

[^9]由于这些函数可能非常频繁，请注意跟踪每个调用的性能开销：应用程序可能会减慢 2 倍或更多。

## A.1 参考文献

\[Gregg 13d\] Gregg, B.，“USE 方法：Linux 性能检查表”，http://www.brendangregg.com/USEmethod/use-linux.html，2013 年首次发布。