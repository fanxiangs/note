
本章介绍扩展 BPF 的 BCC 和 bpftrace 跟踪前端。这些前端提供了一系列性能分析工具，这些工具在前面的章节中使用过。 BPF 技术在第 3 章“操作系统”、第 3.4.4 节“扩展 BPF”中介绍。综上所述，扩展BPF是一个可以为跟踪器提供编程能力的内核执行环境。

对于那些希望更详细地学习一个或多个系统跟踪器的人来说，本章以及第 13 章“perf”和第 14 章“Ftrace”是可选读物。

扩展 BPF 工具可用于回答以下问题：

*   磁盘 I/O 的延迟是多少（以直方图形式表示）？
*   CPU 调度程序延迟是否高到足以导致问题？
*   应用程序是否受到文件系统延迟的影响？
*   正在发生哪些 TCP 会话以及持续时间是多少？
*   哪些代码路径被阻塞以及阻塞了多长时间？

BPF 与其他跟踪器的不同之处在于它是可编程的。它允许对事件执行用户定义的程序，可以执行过滤、保存和检索信息、计算延迟、执行内核聚合和自定义摘要等的程序。虽然其他跟踪器可能需要将所有事件转储到用户空间并对其进行后处理，但 BPF 允许在内核上下文中高效地进行此类处理。这使得创建性能工具变得切实可行，否则这些工具在生产使用中会花费太多开销。

本章有一个主要部分介绍每个推荐的前端。关键部分是：

*   15.1: 密件抄送
    
    *   15.1.1：安装
        
    *   15.1.2：工具覆盖范围
        
    *   15.1.3: 单一用途工具
        
    *   15.1.4: 多用途工具
        
    *   15.1.5：单行代码
        
*   15.2: bpftrace
    
    *   15.2.1：安装
        
    *   15.2.2: 工具
        
    *   15.2.3：单行代码
        
    *   15.2.4: 编程
        
    *   15.2.5：参考
        

从前面章节中的用法来看，BCC 和 bpftrace 之间的差异可能很明显：BCC 适用于复杂的工具，而 bpftrace 适用于临时自定义程序。一些工具在两者中都实现了，如图 15.1 所示。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/15fig01.jpg)

图 15.1 BPF 跟踪前端

表 15.1 总结了 BCC 和 bpftrace 之间的具体差异。

表 15.1 BCC 与 bpftrace

特征

密件抄送

bpftrace

按存储库划分的工具数量

\>80（密件抄送）

\>30（bpftrace）

\>120（bpf-perf-工具书）

工具使用

通常支持复杂选项（`-h`、`-P PID` 等）和参数

通常很简单：没有选项，零个或一个参数

工具文档

手册页、示例文件

手册页、示例文件

编程语言

用户空间：Python、Lua、C 或 C++ 内核空间：C

bpftrace

编程难度

难的

简单的

每个事件的输出类型

任何事物

文本、JSON

摘要类型

任何事物

计数、最小值、最大值、总和、平均值、log2 直方图、线性直方图；通过零个或多个键

图书馆支持

是（例如，Python 导入）

不

平均节目长度[^1]（无评论）

228行

28行

[^1]基于官方存储库和我的 BPF 图书存储库中提供的工具。

BCC 和 bpftrace 都在包括 Facebook 和 Netflix 在内的许多公司使用。 Netflix 默认将它们安装在所有云实例上，并在全云监控和仪表板之后使用它们进行更深入的分析，特别是 \[Gregg 18e\]：

*   BCC：在需要时，在命令行中使用固定工具来分析存储 I/O、网络 I/O 和进程执行。一些 BCC 工具由图形性能仪表板系统自动执行，为调度程序和磁盘 I/O 延迟热图、CPU 外火焰图等提供数据。此外，自定义 BCC 工具始终作为守护进程运行（基于 tcplife(8)），将网络事件记录到云存储以进行流量分析。
    
*   bpftrace：在需要了解内核和应用程序病理学时开发自定义 bpftrace 工具。
    

以下部分介绍了 BCC 工具、bpftrace 工具和 bpftrace 编程。

## 15.1 密件抄送

BPF 编译器集合（或项目和包名称后的“bcc”）是一个开源项目，包含大量高级性能分析工具以及用于构建这些工具的框架。 BCC 由 Brenden Blanco 创建；我帮助其开发并创建了许多跟踪工具。

作为 BCC 工具的示例，biolatency(8) 将磁盘 I/O 延迟的分布显示为二次方直方图，并且可以通过 I/O 标志对其进行细分：

点击这里查看代码图片

\# **biolatency.py -mF**
Tracing block device I/O... Hit Ctrl-C to end.
^C

flags = Priority-Metadata-Read
     msecs               : count     distribution
         0 -> 1          : 90       |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|  
flags = Write
     msecs               : count     distribution
         0 -> 1          : 24       |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|
         2 -> 3          : 0        |                                        |
         4 -> 7          : 8        |\*\*\*\*\*\*\*\*\*\*\*\*\*                           |

flags = ReadAhead-Read
     msecs               : count     distribution
         0 -> 1          : 3031     |\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*|
         2 -> 3          : 10       |                                        |
         4 -> 7          : 5        |                                        |
         8 -> 15         : 3        |                                        |

此输出显示双模型写入分布，以及许多带有“ReadAhead-Read”标志的 I/O。该工具使用 BPF 来汇总内核空间中的直方图以提高效率，因此用户空间组件只需要读取已经汇总的直方图（计数列）并打印它们。

这些 BCC 工具通常在 BCC 存储库中包含使用消息 (`-h`)、手册页和示例文件：

https://github.com/iovisor/bcc

本节总结了 BCC 及其单用途和多用途性能分析工具。

### 15.1.1 安装

BCC 软件包适用于许多 Linux 发行版，包括 Ubuntu、Debian、RHEL、Fedora 和 Amazon Linux，安装非常简单。搜索“bcc-tools”或“bpfcc-tools”或“bcc”（软件包维护者对其进行了不同的命名）。

您还可以从源代码构建 BCC。有关最新的安装和构建说明，请检查 BCC 存储库 \[Iovisor 20b\] 中的 INSTALL.md。 INSTALL.md 还列出了内核配置要求（包括 CONFIG\_BPF=y、CONFIG\_BPF\_SYSCALL=y、CONFIG\_BPF\_EVENTS=y）。 BCC 至少需要 Linux 4.4 才能运行某些工具；对于大多数工具，需要 4.9 或更高版本。

### 15.1.2 工具覆盖范围

BCC 跟踪工具如图 15.2 所示（有些使用通配符进行分组：例如，`java*` 表示所有以“java”开头的工具）。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/15fig02.jpg)

图15.2 密件抄送工具

许多是单一用途的工具，带有单个箭头；有些是左侧列出的多用途工具，并带有双箭头以显示其覆盖范围。

### 15.1.3 单一用途工具

我根据与第 14 章中的性能工具相同的“做一份工作并做好”理念开发了其中的许多工具。这种设计包括使它们的默认输出简洁且通常足够。您可以“仅运行生物潜伏期”，无需学习任何命令行选项，并且通常会获得足够的输出来解决您的问题，而不会造成混乱。通常确实存在用于定制的选项，例如按 I/O 标志进行细分的 biolatency(8) `-F`，如前所述。

表 15.2 描述了一些单一用途工具的选择，包括它们在本书中的位置（如果存在）。请参阅 BCC 存储库以获取完整列表 \[Iovisor 20a\]。

表 15.2 选定的单一用途 BCC 工具

工具

描述

部分

生物潜伏期(8)

将块 I/O（磁盘 I/O）延迟总结为直方图

[[ch09.xhtml#ch09lev6sec6|9.6.6]]

群落生境(8)

按进程汇总块 I/O

[[ch09.xhtml#ch09lev6sec8|9.6.8]]

生物窥探(8)

跟踪块 I/O 的延迟和其他详细信息

[[ch09.xhtml#ch09lev6sec7|9.6.7]]

一口大小(8)

将块 I/O 大小总结为进程直方图

\-

btrfsdist(8)

将 btrfs 操作延迟总结为直方图

[[ch08.xhtml#ch08lev6sec13|8.6.13]]

btrfsslower(8)

跟踪缓慢的 btrfs 操作

[[ch08.xhtml#ch08lev6sec14|8.6.14]]

cpudist(8)

将每个进程的 CPU 开启和关闭时间总结为直方图

[[ch06_split_001.xhtml#ch06lev6sec15|6.6.15]], [[ch16.xhtml#ch16lev1sec7|16.1.7]]

CPU 无人认领(8)

尽管有需求，但仍显示无人认领和空闲的 CPU

\-

临界状态(8)

跟踪长原子关键内核部分

\-

db 减慢(8)

跟踪数据库慢查询

\-

数据库统计(8)

将数据库查询延迟总结为直方图

\-

drsnoop(8)

使用 PID 和延迟跟踪直接内存回收事件

[[ch07.xhtml#ch07lev5sec11|7.5.11]]

执行监听(8)

通过 execve(2) 系统调用跟踪新进程

[[ch01.xhtml#ch01lev7sec3|1.7.3]], [[ch05.xhtml#ch05lev5sec5|5.5.5]]

ext4dist(8)

将 ext4 操作延迟总结为直方图

[[ch08.xhtml#ch08lev6sec13|8.6.13]]

ext4慢速(8)

跟踪缓慢的 ext4 操作

[[ch08.xhtml#ch08lev6sec14|8.6.14]]

文件寿命(8)

追踪短期文件的生命周期

\-

获取主机延迟(8)

通过解析器函数跟踪 DNS 延迟

\-

硬化(8)

总结hardirq事件时间

[[ch06_split_001.xhtml#ch06lev6sec19|6.6.19]]

杀戮窥探(8)

由kill(2)系统调用发出的跟踪信号

\-

时钟状态(8)

总结内核互斥锁统计信息

\-

llcstat(8)

按进程汇总 CPU 缓存引用和未命中情况

\-

内存泄漏(8)

显示未完成的内存分配

\-

mysqld\_qslower(8)

跟踪 MySQL 慢查询

\-

nfsdist(8)

跟踪缓慢的 NFS 操作

[[ch08.xhtml#ch08lev6sec13|8.6.13]]

nfsslower(8)

将 NFS 操作延迟总结为直方图

[[ch08.xhtml#ch08lev6sec14|8.6.14]]

关闭CPU时间(8)

通过堆栈跟踪总结脱离 CPU 的时间

[[ch05.xhtml#ch05lev5sec3|5.5.3]]

下班时间(8)

按 CPU 外堆栈和唤醒堆栈汇总阻塞时间

\-

杀戮(8)

追踪内存不足 (OOM) 杀手

\-

打开监听(8)

跟踪 open(2) 系列系统调用

[[ch08.xhtml#ch08lev6sec10|8.6.10]]

简介(8)

使用堆栈跟踪的定时采样来分析 CPU 使用情况

[[ch05.xhtml#ch05lev5sec2|5.5.2]]

运行qlat(8)

将运行队列（调度程序）延迟总结为直方图

[[ch06_split_001.xhtml#ch06lev6sec16|6.6.16]]

运行队列(8)

使用定时采样总结运行队列长度

[[ch06_split_001.xhtml#ch06lev6sec17|6.6.17]]

运行速度减慢(8)

跟踪长时间运行队列延迟

\-

同步监听(8)

跟踪sync(2)系列系统调用

\-

系统计数(8)

总结系统调用计数和延迟

[[ch05.xhtml#ch05lev5sec6|5.5.6]]

TCP生活(8)

跟踪 TCP 会话并总结其生命周期

[[ch10_split_001.xhtml#ch10lev6sec9|10.6.9]]

tcpretrans(8)

跟踪 TCP 重传，并提供包括内核状态在内的详细信息

[[ch10_split_001.xhtml#ch10lev6sec11|10.6.11]]

TCP 顶部(8)

按主机和 PID 总结 TCP 发送/接收吞吐量

[[ch10_split_001.xhtml#ch10lev6sec10|10.6.10]]

唤醒时间(8)

通过唤醒器堆栈总结睡眠到唤醒时间

\-

xfsdist(8)

将 xfs 操作延迟总结为直方图

[[ch08.xhtml#ch08lev6sec13|8.6.13]]

xfsslower(8)

跟踪缓慢的 xfs 操作

[[ch08.xhtml#ch08lev6sec14|8.6.14]]

兹夫斯分布(8)

将 zfs 操作延迟总结为直方图

[[ch08.xhtml#ch08lev6sec13|8.6.13]]

zfsslower(8)

跟踪缓慢的 zfs 操作

[[ch08.xhtml#ch08lev6sec14|8.6.14]]

有关这些示例，请参阅前面的章节以及 BCC 存储库中的 \*\_example.txt 文件（其中许多文件也是我编写的）。对于本书中未涉及的工具，另请参阅 \[Gregg 19\]。

### 15.1.4 多用途工具

多用途工具列于图 15.2 左侧。它们支持多个事件源并且可以执行许多角色，类似于 perf(1)，尽管这也使得它们使用起来很复杂。表 15.3 对它们进行了描述。

表 15.3 多用途性能工具

工具

描述

部分

参数分布(8)

将函数参数值显示为直方图或计数

15.1.15

函数计数(8)

计算内核或用户级函数调用数

15.1.15

功能较慢(8)

跟踪缓慢的内核或用户级函数调用

\-

功能(8)

将函数延迟总结为直方图

\-

堆栈计数(8)

计算导致事件的堆栈跟踪

15.1.15

痕迹(8)

使用过滤器跟踪任意函数

15.1.15

为了帮助您记住有用的调用，您可以收集俏皮话。我在下一节中提供了一些内容，类似于我的 perf(1) 和trace-cmd 的单行部分。

### 15.1.5 单行代码

除非另有说明，否则以下单行语句将在系统范围内进行跟踪，直到键入 Ctrl-C。它们按工具分组。

#### 函数计数(8)

计算 VFS 内核调用次数：

funcgraph 'vfs\_\*'

计算 TCP 内核调用数：

funccount 'tcp\_\*'

计算每秒 TCP 发送调用数：

funccount -i 1 'tcp\_send\*'

显示每秒块 I/O 事件的速率：

funccount -i 1 't:block:\*'

显示每秒 libc getaddrinfo()（名称解析）的速率：

funccount -i 1 c:getaddrinfo

#### 堆栈计数(8)

计算创建块 I/O 的堆栈跟踪：

点击这里查看代码图片

stackcount t:block:block\_rq\_insert

计算导致发送 IP 数据包的堆栈跟踪，以及负责的 PID：

stackcount -P ip\_output

计算导致线程阻塞并移出 CPU 的堆栈跟踪：

点击这里查看代码图片

stackcount t:sched:sched\_switch

#### 痕迹(8)

使用文件名跟踪内核 do\_sys\_open() 函数：

点击这里查看代码图片

trace 'do\_sys\_open "%s", arg2'

跟踪内核函数 do\_sys\_open() 的返回并打印返回值：

点击这里查看代码图片

trace 'r::do\_sys\_open "ret: %d", retval'

使用模式和用户级堆栈跟踪内核函数 do\_nanosleep()：

点击这里查看代码图片

trace -U 'do\_nanosleep "mode: %d", arg2'

通过 pam 库跟踪身份验证请求：

点击这里查看代码图片

trace 'pam:pam\_start "%s: %s", arg1, arg2'

#### 参数分布(8)

按返回值（大小或错误）总结 VFS 读取：

argdist -H 'r::vfs\_read()'

通过 PID 1005 的返回值（大小或错误）总结 libc read()：

点击这里查看代码图片

argdist -p 1005 -H 'r:c:read()'

按系统调用 ID 计数系统调用：

点击这里查看代码图片

argdist.py -C 't:raw\_syscalls:sys\_enter():int:args->id'

使用计数总结内核函数 tcp\_sendmsg() 大小参数：

点击这里查看代码图片

argdist -C 'p::tcp\_sendmsg(struct sock \*sk, struct msghdr \*msg, size\_t size):u32:size'

将 tcp\_sendmsg() 大小总结为 2 的幂直方图：

点击这里查看代码图片

argdist -H 'p::tcp\_sendmsg(struct sock \*sk, struct msghdr \*msg, size\_t size):u32:size'

按文件描述符计算 PID 181 的 libc write() 调用：

点击这里查看代码图片

argdist -p 181 -C 'p:c:write(int fd):int:fd'

按延迟 >100 μs 的进程汇总读取：

点击这里查看代码图片

argdist -C 'r::\_\_vfs\_read():u32:$PID:$latency > 100000

### 15.1.6 多工具示例

作为使用多功能工具的示例，下面显示了 trace(8) 工具跟踪内核函数 do\_sys\_open() 并将第二个参数打印为字符串：

点击这里查看代码图片

\# **trace 'do\_sys\_open "%s", arg2'**
PID     TID     COMM        FUNC             -
28887   28887   ls          do\_sys\_open      /etc/ld.so.cache
28887   28887   ls          do\_sys\_open      /lib/x86\_64-linux-gnu/libselinux.so.1
28887   28887   ls          do\_sys\_open      /lib/x86\_64-linux-gnu/libc.so.6
28887   28887   ls          do\_sys\_open      /lib/x86\_64-linux-gnu/libpcre2-8.so.0
28887   28887   ls          do\_sys\_open      /lib/x86\_64-linux-gnu/libdl.so.2
28887   28887   ls          do\_sys\_open      /lib/x86\_64-linux-gnu/libpthread.so.0
28887   28887   ls          do\_sys\_open      /proc/filesystems
28887   28887   ls          do\_sys\_open      /usr/lib/locale/locale-archive
\[...\]

跟踪语法受到 printf(3) 的启发，支持格式字符串和参数。在本例中，第二个参数 arg2 被打印为字符串，因为它包含文件名。

Trace(8) 和 argdist(8) 都支持允许创建许多自定义单行代码的语法。以下部分介绍的 bpftrace 更进一步，提供了一种用于编写单行或多行程序的成熟语言。

### 15.1.7 BCC 与 bpftrace

本章开头总结了这些差异。 BCC 适用于支持各种参数或使用各种库的自定义和复杂工具。 bpftrace 非常适合不接受任何参数或单个整数参数的单行工具或短工具。 BCC 允许使用 C 语言开发作为跟踪工具核心的 BPF 程序，从而实现完全控制。这是以复杂性为代价的：BCC 工具的开发时间是 bpftrace 工具的十倍，并且代码行数是 bpftrace 工具的十倍。由于开发工具通常需要多次迭代，因此我发现首先在 bpftrace 中开发工具会节省时间，这样速度更快，然后根据需要将它们移植到 BCC。

BCC 和 bpftrace 之间的区别就像 C 编程和 shell 脚本之间的区别，其中 BCC 就像 C 编程（有些是 C 编程），而 bpftrace 就像 shell 脚本。在我的日常工作中，我使用许多预构建的 C 程序（top(1)、vmstat(1) 等）并开发自定义的一次性 shell 脚本。同样，我还使用许多预构建的 BCC 工具，并开发自定义的一次性 bpftrace 工具。

我在本书中提供了支持这种用法的材料：许多章节展示了您可以使用的 BCC 工具，本章后面的部分展示了如何开发自定义 bpftrace 工具。

### 15.1.8 文档

工具通常有一个使用消息来总结其语法。例如：

点击这里查看代码图片

\# **funccount -h**
usage: funccount \[-h\] \[-p PID\] \[-i INTERVAL\] \[-d DURATION\] \[-T\] \[-r\] \[-D\]
                 pattern

Count functions, tracepoints, and USDT probes

positional arguments:
  pattern               search expression for events

optional arguments:
  -h, --help            show this help message and exit
  -p PID, --pid PID     trace this PID only
  -i INTERVAL, --interval INTERVAL
                        summary interval, seconds
  -d DURATION, --duration DURATION
                        total duration of trace, seconds
  -T, --timestamp       include timestamp on output
  -r, --regexp          use regular expressions. Default is "\*" wildcards
                        only.
  -D, --debug           print BPF program before starting (for debugging
                        purposes)
examples:
    ./funccount 'vfs\_\*'             # count kernel fns starting with "vfs"
    ./funccount -r '^vfs.\*'         # same as above, using regular expressions
    ./funccount -Ti 5 'vfs\_\*'       # output every 5 seconds, with timestamps
    ./funccount -d 10 'vfs\_\*'       # trace for 10 seconds only
    ./funccount -p 185 'vfs\_\*'      # count vfs calls for PID 181 only
    ./funccount t:sched:sched\_fork  # count calls to the sched\_fork tracepoint
    ./funccount -p 185 u:node:gc\*   # count all GC USDT probes in node, PID 185
    ./funccount c:malloc            # count all malloc() calls in libc
    ./funccount go:os.\*             # count all "os.\*" calls in libgo
    ./funccount -p 185 go:os.\*      # count all "os.\*" calls in libgo, PID 185
    ./funccount ./test:read\*        # count "read\*" calls in the ./test binary

每个工具在 bcc 存储库中还有一个手册页 (man/man8/funccount.8) 和一个示例文件 (examples/funccount\_example.txt)。示例文件包含带有注释的输出示例。

我还在 BCC 存储库 \[Iovisor 20b\] 中创建了以下文档：

*   最终用户教程：docs/tutorial.md
    
*   BCC 开发人员教程：docs/tutorial\_bcc\_python\_developer.md
    
*   参考指南：docs/reference\_guide.md
    

我之前书中的第 4 章重点介绍了 BCC \[Gregg 19\]。

## 15.2 bpftrace

bpftrace 是一个基于 BPF 和 BCC 构建的开源跟踪器，它不仅提供了一套性能分析工具，而且还提供了一种高级语言来帮助您开发新的工具。该语言被设计得简单易学。它是跟踪的 awk(1)，并且基于 awk(1)。在 awk(1) 中，您可以编写一个程序节来处理输入行，而使用 bpftrace 您可以编写一个程序节来处理输入事件。 bpftrace 由 Alastair Robertson 创建，我已成为主要贡献者。

作为 bpftrace 的示例，以下一行显示了按进程名称划分的 TCP 接收消息大小的分布：

点击这里查看代码图片

\# **bpftrace -e 'kr:tcp\_recvmsg /retval >= 0/ { @recv\_bytes\[comm\] = hist(retval); }'**
Attaching 1 probe...
^C

@recv\_bytes\[sshd\]:
\[32, 64)               7 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
\[64, 128)              2 |@@@@@@@@@@@@@@                                      |
@recv\_bytes\[nodejs\]:
\[0\]                   82 |@@@@@@@@@@@@@@@@@@@@@@@@@@                          |
\[1\]                  135 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@        |
\[2, 4)               153 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  |
\[4, 8)                12 |@@@                                                 |
\[8, 16)                6 |@                                                   |
\[16, 32)              32 |@@@@@@@@@@                                          |
\[32, 64)             158 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
\[64, 128)            155 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ |
\[128, 256)            14 |@@@@                                                |

此输出显示具有双模式接收大小的 NodeJS 进程，一种模式大约为 0 到 4 字节，另一种模式大约为 32 到 128 字节。

使用简洁的语法，这个 bpftrace 单行代码使用 kretprobe 来检测 tcp\_recvmsg()，当返回值为正时进行过滤（以排除负错误代码），并使用一个名为 `@recv_bytes` 的 BPF 映射对象填充返回值的直方图，使用进程名称 (`comm`) 作为键保存。当键入 Ctrl-C 并且 bpftrace 收到信号 (SIGINT) 时，它结束并自动打印出 BPF 映射。以下各节将更详细地解释此语法。

除了使您能够编写自己的单行代码之外，bpftrace 在其存储库中还附带了许多现成运行的工具：

https://github.com/iovisor/bpftrace

本节总结了 bpftrace 工具和 bpftrace 编程语言。这是基于我在 \[Gregg 19\] 中的 bpftrace 材料，该材料更深入地探讨了 bpftrace。

### 15.2.1 安装

bpftrace 软件包可用于许多 Linux 发行版，包括 Ubuntu，因此安装变得非常简单。搜索名为“bpftrace”的包；它们适用于 Ubuntu、Fedora、Gentoo、Debian、OpenSUSE 和 CentOS。 RHEL 8.2 将 bpftrace 作为技术预览。

除了软件包之外，还有 bpftrace 的 Docker 镜像、除了 glibc 之外不需要依赖项的 bpftrace 二进制文件，以及从源代码构建 bpftrace 的说明。有关这些选项的文档，请参阅 bpftrace 存储库 \[Iovisor 20a\] 中的 INSTALL.md，其中还列出了内核要求（包括 CONFIG\_BPF=y、CONFIG\_BPF\_SYSCALL=y、CONFIG\_BPF\_EVENTS=y）。 bpftrace 需要 Linux 4.9 或更高版本。

### 15.2.2 工具

bpftrace 跟踪工具如图 15.3 所示。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/15fig03.jpg)

图15.3 bpftrace工具

bpftrace 存储库中的工具以黑色显示。在我之前的书中，我开发了更多 bpftrace 工具，并将它们作为开源发布在 bpf-perf-tools-book 存储库中：它们以红色/灰色显示 \[Gregg 19g\]。

### 15.2.3 单行代码

除非另有说明，否则以下单行语句将在系统范围内进行跟踪，直到键入 Ctrl-C。除了其固有的用途之外，它们还可以作为 bpftrace 编程语言的迷你示例。这些按目标分组。可以在每个资源章节中找到更长的 bpftrace 单行代码列表。

#### 中央处理器

使用参数跟踪新进程：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_execve { join(args->argv); }'

按进程计数系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:raw\_syscalls:sys\_enter { @\[pid, comm\] = count(); }'

PID 189、49 赫兹的用户级堆栈示例：

点击这里查看代码图片

bpftrace -e 'profile:hz:49 /pid == 189/ { @\[ustack\] = count(); }'

#### 记忆

按代码路径计算进程堆扩展（brk()）：

点击这里查看代码图片

bpftrace -e tracepoint:syscalls:sys\_enter\_brk { @\[ustack, comm\] = count(); }

通过用户级堆栈跟踪来计算用户页面错误：

点击这里查看代码图片

bpftrace -e 'tracepoint:exceptions:page\_fault\_user { @\[ustack, comm\] =
    count(); }'

按跟踪点对 vmscan 操作进行计数：

点击这里查看代码图片

bpftrace -e 'tracepoint:vmscan:\* { @\[probe\]++; }'

#### 文件系统

通过 openat(2) 打开的跟踪文件，进程名称为：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_openat { printf("%s %s\\n", comm,
    str(args->filename)); }'

显示 read() 系统调用读取字节（和错误）的分布：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_exit\_read { @ = hist(args->ret); }'

计算 VFS 调用数：

点击这里查看代码图片

bpftrace -e 'kprobe:vfs\_\* { @\[probe\] = count(); }'

统计 ext4 跟踪点调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:ext4:\* { @\[probe\] = count(); }'

#### 磁盘

将块 I/O 大小总结为直方图：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @bytes = hist(args->bytes); }'

计算块 I/O 请求用户堆栈跟踪：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @\[ustack\] = count(); }'

计算块 I/O 类型标志：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @\[args->rwbs\] = count(); }'

#### 联网

通过 PID 和进程名称对套接字接受(2)进行计数：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_accept\* { @\[pid, comm\] = count(); }'

通过 CPU 上的 PID 和进程名称来计算套接字发送/接收字节数：

点击这里查看代码图片

bpftrace -e 'kr:sock\_sendmsg,kr:sock\_recvmsg /retval > 0/ {
    @\[pid, comm\] = sum(retval); }'

TCP 以直方图形式发送字节：

点击这里查看代码图片

bpftrace -e 'k:tcp\_sendmsg { @send\_bytes = hist(arg2); }'

TCP 接收字节作为直方图：

点击这里查看代码图片

bpftrace -e 'kr:tcp\_recvmsg /retval >= 0/ { @recv\_bytes = hist(retval); }'

UDP 以直方图形式发送字节：

点击这里查看代码图片

bpftrace -e 'k:udp\_sendmsg { @send\_bytes = hist(arg2); }'

#### 应用领域

通过用户堆栈跟踪求和 malloc() 请求的字节（高开销）：

点击这里查看代码图片

bpftrace -e 'u:/lib/x86\_64-linux-gnu/libc-2.27.so:malloc { @\[ustack(5)\] =
    sum(arg0); }'

跟踪kill()信号显示发送者进程名称、目标PID和信号号：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_kill { printf("%s -> PID %d SIG %d\\n",
    comm, args->pid, args->sig); }'

#### 核心

按系统调用函数计数系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:raw\_syscalls:sys\_enter {
    @\[ksym(\*(kaddr("sys\_call\_table") + args->id \* 8))\] = count(); }'

计算以“attach”开头的内核函数调用：

点击这里查看代码图片

bpftrace -e 'kprobe:attach\* { @\[probe\] = count(); }'

频率计数 vfs\_write() 的第三个参数（大小）：

点击这里查看代码图片

bpftrace -e 'kprobe:vfs\_write { @\[arg2\] = count(); }'

对内核函数 vfs\_read() 进行计时并总结为直方图：

点击这里查看代码图片

bpftrace -e 'k:vfs\_read { @ts\[tid\] = nsecs; } kr:vfs\_read /@ts\[tid\]/ {
    @ = hist(nsecs - @ts\[tid\]); delete(@ts\[tid\]); }'

计算上下文切换堆栈跟踪：

点击这里查看代码图片

bpftrace -e 't:sched:sched\_switch { @\[kstack, ustack, comm\] = count(); }'

99 赫兹的内核级堆栈示例（不包括空闲）：

点击这里查看代码图片

bpftrace -e 'profile:hz:99 /pid/ { @\[kstack\] = count(); }'

### 15.2.4 编程

本节提供有关使用 bpftrace 以及使用 bpftrace 语言进行编程的简短指南。本节的格式受到 awk \[Aho 78\]\[Aho 88\] 原始论文的启发，该论文用六页内容介绍了该语言。 bpftrace 语言本身受到 awk 和 C 以及包括 DTrace 和 SystemTap 在内的跟踪器的启发。

以下是 bpftrace 编程的示例：它测量 vfs\_read() 内核函数中的时间并将该时间（以微秒为单位）打印为直方图。

点击这里查看代码图片

#!/usr/local/bin/bpftrace

// this program times vfs\_read()

kprobe:vfs\_read
{
        @start\[tid\] = nsecs;
}

kretprobe:vfs\_read
/@start\[tid\]/
{
        $duration\_us = (nsecs - @start\[tid\]) / 1000;
        @us = hist($duration\_us);
        delete(@start\[tid\]);
}

以下部分解释了该工具的组件，可以将其视为教程。第 15.2.5 节“参考”是参考指南摘要，包括探针类型、测试、运算符、变量、函数和映射类型。

#### 1\. 使用方法

命令

bpftrace -e program

将执行该程序，检测它定义的任何事件。该程序将运行直到 Ctrl-C，或者直到它显式调用 `exit()`。作为 `-e` 参数运行的 bpftrace 程序称为单行程序。或者，可以将程序保存到文件并使用以下命令执行：

bpftrace file.bt

.bt 扩展名不是必需的，但有助于以后识别。通过在文件顶部放置解释行[^2]

[^2]有些人更喜欢使用 #!/usr/bin/env bpftrace，这样就可以从 $PATH 找到 bpftrace。然而，env(1) 存在各种问题，并且它在其他项目中的使用已被恢复。

#!/usr/local/bin/bpftrace

该文件可以成为可执行文件（`chmod a+x file.bt`）并像任何其他程序一样运行：

./file.bt

bpftrace 必须由 root 用户（超级用户）执行。[^3] 对于某些环境，可以使用 root shell 直接执行程序，而其他环境可能更喜欢通过 sudo 运行特权命令(1 ）：

sudo ./file.bt

[^3]bpftrace 检查 UID 0；未来的更新可能会检查特定权限。

#### 2\. 程序结构

bpftrace 程序是一系列具有关联操作的探测器：

probes { actions }
probes { actions }
...

当探测器触发时，将执行相关操作。操作之前可以包含可选的过滤表达式：

probes /filter/ { actions }

仅当过滤器表达式为 true 时才会触发该操作。这类似于 awk(1) 程序结构：

/pattern/ { actions }

awk(1) 编程也类似于 bpftrace 编程：可以定义多个操作块，它们可以按任何顺序执行，当它们的模式或探针 + 过滤器表达式为 true 时触发。

#### 3\. 评论

对于 bpftrace 程序文件，可以添加带有“`//`”前缀的单行注释：

// this is a comment

这些评论不会被执行。多行注释使用与C中相同的格式：

/\*
 \* This is a
 \* multi-line comment.
 \*/

此语法也可用于部分行注释（例如，`/* comment */`）。

#### 4\. 探针格式

探针以探针类型名称开始，然后是冒号分隔的标识符的层次结构：

点击这里查看代码图片

type:identifier1\[:identifier2\[...\]\]

层次结构由探针类型定义。考虑这两个例子：

kprobe:vfs\_read
uprobe:/bin/bash:readline

kprobe 探测类型检测内核函数调用，并且只需要一个标识符：内核函数名称。 uprobe 探测类型检测用户级函数调用，并且需要二进制文件的路径和函数名称。

可以使用逗号分隔符指定多个探测器来执行相同的操作。例如：

probe1,probe2,... { actions }

有两种不需要额外标识符的特殊探测类型：BEGIN 和 END 分别触发 bpftrace 程序的开头和结尾（就像 awk(1) 一样）。例如，要在跟踪开始时打印信息性消息：

点击这里查看代码图片

BEGIN { printf("Tracing. Hit Ctrl-C to end.\\n"); }

要了解有关探针类型及其用法的更多信息，请参阅第 15.2.5 节“参考”标题下的 1. 探针类型。

#### 5\. 探测通配符

某些探针类型接受通配符。探头

kprobe:vfs\_\*

将检测所有以“vfs\_”开头的 kprobes（内核函数）。

检测过多的探针可能会产生不必要的性能开销。为了避免意外发生此问题，bpftrace 有一个可调节的最大探测器数量，可通过 BPFTRACE\_MAX\_PROBES 环境变量设置（当前默认为 512[^4]）。

[^4]目前超过 512 个会导致 bpftrace 启动和关闭速度变慢，因为它会逐个检测它们。未来的内核工作计划批量探测仪器。到那时，这个限制就可以大大增加，甚至取消。

您可以在使用通配符之前通过运行 `bpftrace -l` 列出匹配的探针来测试通配符：

点击这里查看代码图片

\# **bpftrace -l 'kprobe:vfs\_\*'**
kprobe:vfs\_fallocate
kprobe:vfs\_truncate
kprobe:vfs\_open
kprobe:vfs\_setpos
kprobe:vfs\_llseek
\[…\]
bpftrace -l 'kprobe:vfs\_\*' | wc -l
56

这匹配了 56 个探针。探针名称用引号引起来，以防止意外的 shell 扩展。

#### 6\. 过滤器

过滤器是布尔表达式，用于控制是否执行操作。过滤器

/pid == 123/

仅当内置 pid（进程 ID）等于 123 时才会执行该操作。

如果未指定测试

/pid/

过滤器将检查内容是否非零（`/pid/` 与 `/pid != 0/` 相同）。过滤器可以与布尔运算符组合，例如逻辑 AND (`&&`)。例如：

/pid > 100 && pid < 1000/

这要求两个表达式的计算结果均为“true”。

#### 7\. 行动

一个操作可以是单个语句或用分号分隔的多个语句：

点击这里查看代码图片

{ action one; action two; action three }

最后的语句还可以附加分号。语句采用bpftrace语言编写，类似于C语言，可以操作变量并执行bpftrace函数调用。例如，动作

点击这里查看代码图片

{ $x = 42; printf("$x is %d", $x); }

将变量 `$x` 设置为 42，然后使用 `printf()` 打印它。有关其他可用函数调用的摘要，请参阅第 15.2.5 节“参考”，标题为 4. 函数和 5. 映射函数。

#### 8\. 世界，你好！

您现在应该了解以下基本程序，该程序打印“Hello, World!”当 bpftrace 开始运行时：

点击这里查看代码图片

\# bpftrace -e 'BEGIN { printf("Hello, World!\\n"); }'
Attaching 1 probe...
Hello, World!
^C

作为一个文件，它可以被格式化为：

#!/usr/local/bin/bpftrace

BEGIN
{
        printf("Hello, World!\\n");
}

使用缩进的操作块跨越多行是不必要的，但它提高了可读性。

#### 9 功能

除了用于打印格式化输出的 `printf()` 之外，其他内置函数还包括：

*   `exit()`：退出 bpftrace
    
*   `str(char *)`：从指针返回字符串
    
*   `system(format[, arguments ...])`：在 shell 中运行命令
    

行动

点击这里查看代码图片

printf("got: %llx %s\\n", $x, str($x)); exit();

会将 `$x` 变量打印为十六进制整数，然后将其视为以 NULL 结尾的字符数组指针 (char \*) 并将其打印为字符串，然后退出。

#### 10 个变量

共有三种变量类型：内置变量、暂存变量和映射变量。

内置变量是由 bpftrace 预定义和提供的，通常是只读信息源。它们包括表示进程 ID 的 `pid`、表示进程名称的 `comm`、表示纳秒时间戳的 `nsecs` 和表示当前线程的 task\_struct 地址的 `curtask`。

临时变量可用于临时计算，并具有前缀“`”。他们的名字和类型是在他们的第一次任务中设置的。声明：

点击这里查看代码图片

$x = 1;
$y = “hello”;
$z = (struct task\_struct \*)curtask;

将 `$x` 声明为整数，将 `$y` 声明为字符串，将 `$z` 声明为指向 struct task\_struct 的指针。这些变量只能在分配它们的操作块中使用。如果在没有赋值的情况下引用变量，bpftrace 会打印错误（这可以帮助您捕获拼写错误）。

映射变量使用 BPF 映射存储对象并具有前缀“`@`”。它们可用于全局存储，在操作之间传递数据。该计划：

probe1 { @a = 1; }
probe2 { $x = @a; }

当探针 1 触发时，将 1 分配给 `@a`；当探针 2 触发时，将 `@a` 分配给 `$x`。如果先触发probe1，然后再触发probe2，则`$x`将设置为1；否则为 0（未初始化）。

可以使用映射作为哈希表（关联数组）为键提供一个或多个元素。声明

@start\[tid\] = nsecs;

经常使用：将 `nsecs` 内置函数分配给名为 `@start` 的映射，并以当前线程 ID `tid` 为键。这允许线程存储不会被其他线程覆盖的自定义时间戳。

@path\[pid, $fd\] = str(arg0);

是一个多键映射的示例，其中使用 `pid` 内置函数和 `$fd` 变量作为键。

#### 11 地图功能

地图可以分配给特殊功能。这些函数以自定义方式存储和打印数据。作业

@x = count();

对事件进行计数，并在打印时打印计数。这使用了每 CPU 映射，并且 `@x` 成为计数类型的特殊对象。以下语句也对事件进行计数：

@x++;

但是，这使用全局 CPU 映射（而不是每个 CPU 映射）来提供 `@x` 作为整数。对于某些需要整数而不是计数的程序来说，这种全局整数类型有时是必要的，但请记住，由于并发更新可能会出现较小的误差范围。

作业

@y = sum($x);

对 `$x` 变量求和，并在打印时打印总数。作业

@z = hist($x);

将 `$x` 存储在 2 的幂直方图中，打印时将打印存储桶计数和 ASCII 直方图。

某些地图功能直接在地图上运行。例如：

print(@x);

将打印 `@x` 地图。例如，这可用于打印间隔事件上的地图内容。这并不经常使用，因为为了方便起见，当 bpftrace 终止时，所有地图都会自动打印。[^5]

[^5]当 bpftrace 终止时，打印地图的开销也会减少，因为在运行时地图正在经历更新，这会减慢地图遍历例程的速度。

某些地图功能在地图键上运行。例如：

delete(@start\[tid\]);

从 `@start` 映射中删除键为 `tid` 的键值对。

#### 12\. 定时vfs\_read()

您现在已经学习了理解更复杂和更实际的示例所需的语法。这个程序 vfsread.bt 对 vfs\_read 内核函数进行计时，并打印出其持续时间的直方图（以微秒 (us) 为单位）：

点击这里查看代码图片

#!/usr/local/bin/bpftrace

// this program times vfs\_read()

kprobe:vfs\_read
{
        @start\[tid\] = nsecs;
}

kretprobe:vfs\_read
/@start\[tid\]/
{
        $duration\_us = (nsecs - @start\[tid\]) / 1000;
        @us = hist($duration\_us);
        delete(@start\[tid\]);
}

通过使用 kprobe 检测其开始并将时间戳存储在以线程 ID 为键的 `@start` 哈希中，然后使用 kretprobe 检测其结束并计算增量，计算 vfs\_read() 内核函数的持续时间：现在-开始。使用过滤器来确保记录开始时间；否则，对于跟踪开始时正在进行的 vfs\_read() 调用，增量计算将变得虚假，因为可以看到结束但看不到开始（增量将变为：now - 0）。

示例输出：

点击这里查看代码图片

\# **bpftrace vfsread.bt**
Attaching 2 probes...
^C

@us:
\[0\]                   23 |@                                                   |
\[1\]                  138 |@@@@@@@@@                                           |
\[2, 4)               538 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@               |
\[4, 8)               744 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|
\[8, 16)              641 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@        |
\[16, 32)             122 |@@@@@@@@                                            |
\[32, 64)              13 |                                                    |
\[64, 128)             17 |@                                                   |
\[128, 256)             2 |                                                    |
\[256, 512)             0 |                                                    |
\[512, 1K)              1 |                                                    |

程序一直运行，直到输入 Ctrl-C；然后它打印此输出并终止。该直方图被命名为“us”，作为在输出中包含单位的一种方式，因为地图名称是打印出来的。通过为映射提供有意义的名称（例如“bytes”和“latency\_ns”），您可以注释输出并使其不言自明。

该脚本可以根据需要进行定制。考虑将 `hist()` 赋值行更改为：

点击这里查看代码图片

@us\[pid, comm\] = hist($duration\_us);

这会为每个进程 ID 和进程名称对存储一个直方图。使用传统的系统工具，如 iostat(1) 和 vmstat(1)，输出是固定的，无法轻松定制。但使用 bpftrace，您看到的指标可以进一步分解为多个部分，并使用其他探测器的指标进行增强，直到您获得所需的答案。

请参阅第 8 章“文件系统”、第 8.6.15 节“BPFtrace”、标题“VFS 延迟跟踪”，了解按类型分解 vfs\_read() 延迟的扩展示例：文件系统、套接字等。

### 15.2.5 参考

下面总结了 bpftrace 编程的主要组成部分：探针类型、流控制、变量、函数和映射函数。

#### 1\. 探头类型

表 15.4 列出了可用的探头类型。其中许多还具有快捷方式别名，这有助于创建更短的单行文字。

表 15.4 bpftrace 探针类型

类型

捷径

描述

`tracepoint`

`t`

内核静态检测点

`usdt`

`U`

用户级静态定义的跟踪

`kprobe`

`k`

内核动态函数检测

`kretprobe`

`kr`

内核动态函数返回检测

`kfunc`

`f`

内核动态函数检测（基于 BPF）

`kretfunc`

`fr`

内核动态函数返回检测（基于 BPF）

`uprobe`

`u`

用户级动态函数检测

`uretprobe`

`ur`

用户级动态函数返回检测

`software`

`s`

基于内核软件的事件

`hardware`

`h`

基于硬件计数器的仪器

`watchpoint`

`w`

内存观察点检测

`profile`

`p`

跨所有 CPU 的定时采样

`interval`

`i`

定时报告（来自一个CPU）

`BEGIN`

bpftrace 开始

`END`

bpftrace 结束

大多数这些探针类型都是现有内核技术的接口。第 4 章解释了这些技术的工作原理：kprobes、uprobes、tracepoint、USDT 和 PMC（由硬件探针类型使用）。 kfunc/kretfunc 探针类型是基于 eBPF 蹦床和 BTF 的新型低开销接口。

某些探测器可能会频繁触发，例如针对调度程序事件、内存分配和网络数据包。为了减少开销，请尝试尽可能使用频率较低的事件来解决问题。如果您不确定探测频率，可以使用 bpftrace 进行测量。例如，仅计算 vfs\_read() kprobe 调用一秒钟：

点击这里查看代码图片

\# **bpftrace -e 'k:vfs\_read { @ = count(); } interval:s:1 { exit(); }'**

我选择了较短的持续时间，以尽量减少管理费用（以防管理费用很大）。我认为的高或低频率取决于您的 CPU 速度、数量、余量以及探针仪器的成本。作为当今计算机的粗略指南，我认为每秒少于 100k kprobe 或跟踪点事件属于低频。

##### 探究论点

每种探测类型都提供不同类型的参数，以进一步了解事件的上下文。例如，跟踪点使用 `args` 数据结构中的字段名称提供格式文件中的字段。例如，以下仪器 syscalls:sys\_enter\_read 跟踪点并使用 `args->count` 参数记录 count 参数的直方图（请求的大小）：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_read { @req\_bytes = hist(args->count); }'

这些字段可以从 /sys 中的格式文件或使用 `-lv` 的 bpftrace 中列出：

点击这里查看代码图片

\# **bpftrace -lv 'tracepoint:syscalls:sys\_enter\_read'**
tracepoint:syscalls:sys\_enter\_read
    int \_\_syscall\_nr;
    unsigned int fd;
    char \* buf;
    size\_t count;

有关每种探测类型及其参数的说明，请参阅在线“bpftrace 参考指南”\[Iovisor 20c\]。

#### 2\. 流量控制

bpftrace 中有三种类型的测试：过滤器、三元运算符和 if 语句。这些测试根据布尔表达式有条件地更改程序流程，这些表达式支持表 15.5 中所示的内容。

表 15.5 bpftrace 布尔表达式

表达

描述

**`==`**

等于

**`!=`**

不等于

**`>`**

大于

**`<`**

少于

**`>=`**

大于或等于

**`<=`**

小于或等于

**`&&`**

和

**`||`**

包含或

可以使用括号对表达式进行分组。

##### 筛选

前面介绍过，这些门控是否执行某个操作。格式：

probe /filter/ { action }

可以使用布尔运算符。仅当内置的 `pid` 等于 123 时，过滤器 `/pid == 123/` 才会执行操作。

##### 三元运算符

三元运算符是由一个测试和两个结果组成的三元素运算符。格式：

点击这里查看代码图片

test ? true\_statement : false\_statement

例如，您可以使用三元运算符来查找 `$x` 的绝对值：

$abs = $x >= 0 ? $x : - $x;

##### 如果语句

If 语句具有以下语法：

点击这里查看代码图片

if (test) { true\_statements }
if (test) { true\_statements } else { false\_statements }

一种用例是程序在 IPv4 上执行与 IPv6 上不同的操作。例如（为简单起见，这忽略了 IPv4 和 IPv6 以外的系列）：

点击这里查看代码图片

if ($inet\_family == $AF\_INET) {
    // IPv4
    ...
} else {
    // assume IPv6
    ...
}

自 bpftrace v0.10.0 起支持 `else if` 语句。[^6]

[^6]感谢徐丹尼尔 (PR#1211)。

##### 循环

bpftrace 支持使用 `unroll()` 展开循环。对于 Linux 5.3 及更高版本的内核，还支持 `while()` 循环[^7]：

[^7]感谢 Bas Smit 添加 bpftrace 逻辑 (PR#1066)。

while (test) {
    statements
}

这使用了 Linux 5.3 中添加的内核 BPF 循环支持。

##### 运营商

前面的部分列出了用于测试的布尔运算符。 bpftrace 还支持表 15.6 中所示的运算符。

表 15.6 bpftrace 运算符

操作员

描述

`=`

任务

**`+`**, **`-`**, **`*`**, **`/`**

加法、减法、乘法、除法（仅限整数）

**`++`**, **`--`**

自动递增、自动递减

**`&`**, **`|`**, **`^`**

二进制与、二进制或、二进制异或

`!`

逻辑非

**`<<`**, **`>>`**

逻辑左移、逻辑右移

**`+=`**, **`-=`**, **`*=`**, **`/=`**, **`%=`**, **`&=`**, **`^=`**, **`<<=`**, **`>>=`**

复合运算符

这些运算符是根据 C 编程语言中的类似运算符建模的。

#### 3\. 变量

bpftrace 提供的内置变量通常用于信息的只读访问。表 15.7 列出了重要的内置变量。

表15.7 bpftrace选择的内置变量

内置变量

类型

描述

`pid`

整数

进程 ID（内核 tgid）

`tid`

整数

线程 ID（内核 pid）

`uid`

整数

用户身份

`username`

细绳

用户名

`nsecs`

整数

时间戳，以纳秒为单位

`elapsed`

整数

自 bpftrace 初始化以来的时间戳（以纳秒为单位）

`cpu`

整数

处理器ID

`comm`

细绳

进程名称

`kstack`

细绳

内核堆栈跟踪

`ustack`

细绳

用户级堆栈跟踪

`arg0, ..., argN`

整数

某些探针类型的参数

`args`

结构体

某些探针类型的参数

`sarg0, ..., sargN`

整数

某些探测类型的基于堆栈的参数

`retval`

整数

某些探针类型的返回值

`func`

细绳

跟踪函数的名称

`probe`

细绳

当前探头的全名

`curtask`

结构体/整数

内核task\_struct（作为task\_struct或无符号64位整数，取决于类型信息的可用性）

`cgroup`

整数

当前进程的默认 cgroup v2 ID（用于与 cgroupid() 进行比较）

`$1, ..., $N`

整数，字符\*

bpftrace 程序的位置参数

当前所有整数均为 uint64。这些变量都指的是探测器触发时当前运行的线程、探测器、函数和 CPU。

本章前面已经演示了各种内置函数：`retval`、`comm`、`tid` 和 `nsecs`。有关内置变量的完整更新列表 \[Iovisor 20c\]，请参阅在线“bpftrace 参考指南”。

#### 4\. 功能

表 15.8 列出了为各种任务选择的内置函数。其中一些已在前面的示例中使用过，例如 `printf()`。

表15.8 bpftrace选择的内置函数

功能

描述

`printf(char *fmt [, ...])`

格式化打印

`time(char *fmt)`

打印格式化时间

`join(char *arr[])`

打印由空格字符连接的字符串数组

`str(char *s [, int len])`

从指针 s 返回字符串，具有可选的长度限制

`buf(void *d [, int length])`

返回数据指针的十六进制字符串版本

`strncmp(char *s1, char *s2, int length)`

比较两个字符串的最大长度字符

`sizeof(expression)`

返回表达式或数据类型的大小

`kstack([int limit])`

返回内核堆栈，直至限制帧深度

`ustack([int limit])`

返回最多限制帧深度的用户堆栈

`ksym(void *p)`

解析内核地址并返回字符串符号

`usym(void *p)`

解析用户空间地址并返回字符串符号

`kaddr(char *name)`

将内核符号名称解析为地址

`uaddr(char *name)`

将用户空间符号名称解析为地址

`reg(char *name)`

返回存储在指定寄存器中的值

`ntop([int af,] int addr)`

返回 IPv4/IPv6 地址的字符串表示形式。

`cgroupid(char *path)`

返回给定路径的 cgroup ID (/sys/fs/cgroup/...)

`system(char *fmt [, ...])`

执行 shell 命令

`cat(char *filename)`

打印文件的内容

`signal(char[] sig | u32 sig)`

向当前任务发送信号（例如，SIGTERM）

`override(u64 rc)`

覆盖 kprobe 返回值[^8]

`exit()`

退出 bpftrace

[^8]警告：仅当您知道自己在做什么时才使用此选项：一个小错误可能会导致内核崩溃或损坏。

其中一些函数是异步的：内核将事件排队，稍后在用户空间中对其进行处理。异步函数为 `printf()`、`time()`、`cat()`、`join()` 和 `system()`。函数 `kstack()`、`ustack()`、`ksym()` 和 `usym()` 同步记录地址，但异步进行符号转换。

例如，下面使用 `printf()` 和 `str()` 函数来显示 openat(2) 系统调用的文件名：

点击这里查看代码图片

\# **bpftrace -e 't:syscalls:sys\_enter\_open { printf("%s %s\\n", comm,**
    **str(args->filename)); }'**
Attaching 1 probe...
top /etc/ld.so.cache
top /lib/x86\_64-linux-gnu/libprocps.so.7
top /lib/x86\_64-linux-gnu/libtinfo.so.6
top /lib/x86\_64-linux-gnu/libc.so.6
\[...\]

请参阅在线“bpftrace 参考指南”，了解完整且更新的函数列表 \[Iovisor 20c\]。

#### 5\. 地图功能

映射是 BPF 中的特殊哈希表存储对象，可用于不同目的，例如，作为存储键/值对或用于统计摘要的哈希表。 bpftrace 提供了用于地图分配和操作的内置函数，主要用于支持统计摘要地图。表 15.9 列出了最重要的映射函数。

表15.9 bpftrace选择的映射函数

功能

描述

`count()`

计数出现次数

`sum(int n)`

对值求和

`avg(int n)`

平均该值

`min(int n)`

记录最小值

`max(int n)`

记录最大值

`stats(int n)`

返回计数、平均值和总计

`hist(int n)`

打印值的二次方直方图

`lhist(int n, const int min, const int max, int step)`

打印值的线性直方图

`delete(@m[key])`

删除映射键/值对

`print(@m [, top [, div]])`

打印地图，带有可选限制和除数

`clear(@m)`

从地图中删除所有键

`zero(@m)`

将所有地图值设置为零

其中一些函数是异步的：内核将事件排队，稍后在用户空间中对其进行处理。异步操作为 `print()`、`clear()` 和 `zero()`。当您编写程序时，请记住这种延迟。

作为使用映射函数的另一个示例，下面使用 `lhist()` 按进程名称创建系统调用 read(2) 大小的线性直方图，步长为 1，以便可以独立查看每个文件描述符编号：

点击这里查看代码图片

\# **bpftrace -e 'tracepoint:syscalls:sys\_enter\_read {**
    **@fd\[comm\] = lhist(args->fd, 0, 100, 1); }'**
Attaching 1 probe...
^C
\[...\]
@fd\[sshd\]:
\[4, 5)                22 |                                                    |
\[5, 6)                 0 |                                                    |
\[6, 7)                 0 |                                                    |
\[7, 8)                 0 |                                                    |
\[8, 9)                 0 |                                                    |
\[9, 10)                0 |                                                    |
\[10, 11)               0 |                                                    |
\[11, 12)               0 |                                                    |
\[12, 13)            7760 |@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@|

输出显示，在此系统上 sshd 进程通常从文件描述符 12 读取。输出使用集合表示法，其中“`[`”表示 >=，“`)`”表示 <（又名有界左-闭区间、右开区间）。

请参阅在线“bpftrace 参考指南”，了解完整且更新的地图函数列表 \[Iovisor 20c\]。

### 15.2.6 文档

本书前面的章节中有更多 bpftrace，具体如下：

*   第 5 章，应用，第 5.5.7 节
    
*   第 6 章，CPU，第 6.6.20 节
    
*   第 7 章，内存，第 7.5.13 节
    
*   第 8 章，文件系统，第 8.6.15 节
    
*   第 9 章，磁盘，第 9.6.11 节
    
*   第 10 章，网络，第 10.6.12 节
    

第 4 章“可观测性工具”和第 11 章“云计算”中也有 bpftrace 示例。

在 bpftrace 存储库中，我还创建了以下文档：

*   参考指南：docs/reference\_guide.md \[Iovisor 20c\]
    
*   教程 docs/tutorial\_one\_liners.md \[Iovisor 20d\]
    

有关 bpftrace 的更多信息，请参阅我之前的书 BPF Performance Tools \[Gregg 19\]，其中第 5 章 bpftrace 通过许多示例探讨了编程语言，后面的章节提供了更多用于分析不同目标的 bpftrace 程序。

请注意，\[Gregg 19\] 中描述为“计划”的一些 bpftrace 功能已添加到 bpftrace 中并包含在本章中。它们是：while() 循环、else-if 语句、signal()、override() 和观察点事件。 bpftrace 中添加的其他功能包括 kfunc 探针类型、buf() 和 sizeof()。查看 bpftrace 存储库中的发行说明以了解未来的新增功能，尽管计划中的新增功能并不多：bpftrace 已经为 120 多个已发布的 bpftrace 工具提供了足够的功能。

## 15.3 参考文献

\[Aho 78\] Aho, A. V.、Kernighan, B. W. 和 Weinberger, P. J.，“Awk：模式扫描和处理语言（第二版）”，Unix 第 7 版手册页，1978 年。在线：http://plan9.bell- labs.com/7thEdMan/index.html。

\[Aho 88\] Aho, A. V.、Kernighan, B. W. 和 Weinberger, P. J.，AWK 编程语言，Addison Wesley，1988。

\[Gregg 18e\] Gregg, B.，“哟！ Netflix 2018 年云性能根本原因分析”，http://www.brendangregg.com/blog/2019-04-26/yow2018-cloud-performance-netflix.html，2018 年。

\[Gregg 19\] Gregg, B.，BPF 性能工具：Linux 系统和应用程序可观察性，Addison-Wesley，2019 年。

\[Gregg 19g\] Gregg, B.，“BPF 性能工具（书籍）：工具”，http://www.brendangregg.com/bpf-performance-tools-book.html#tools，2019。

\[Iovisor 20a\]“bpftrace：Linux eBPF 的高级跟踪语言”，https://github.com/iovisor/bpftrace，最后更新于 2020 年。

\[Iovisor 20b\]“BCC - 用于基于 BPF 的 Linux IO 分析、网络、监控等的工具”，https://github.com/iovisor/bcc，最后更新于 2020 年。

\[Iovisor 20c\]“bpftrace 参考指南”，https://github.com/iovisor/bpftrace/blob/master/docs/reference\_guide.md，最后更新于 2020 年。

\[Iovisor 20d\] Gregg, B. 等人，“bpftrace One-Liner 教程”，https://github.com/iovisor/bpftrace/blob/master/docs/tutorial\_one\_liners.md，最后更新于 2020 年。