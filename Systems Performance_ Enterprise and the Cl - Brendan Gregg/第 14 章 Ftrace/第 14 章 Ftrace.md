## 第14章

追踪

Ftrace 是官方的 Linux 跟踪器，是一个由不同跟踪实用程序组成的多功能工具。 Ftrace 由 Steven Rostedt 创建，并首次添加到 Linux 2.6.27 (2008) 中。它无需任何额外的用户级前端即可使用，因此特别适合存储空间非常宝贵的嵌入式 Linux 环境。它对于服务器环境也很有用。

对于那些希望更详细地学习一个或多个系统跟踪器的人来说，本章以及第 13 章“perf”和第 15 章“BPF”是可选读物。

Ftrace 可用于回答以下问题：

*   某些内核函数被调用的频率是多少？
    
*   什么代码路径导致调用该函数？
    
*   该内核函数调用了哪些子函数？
    
*   禁用抢占的代码路径导致的最高延迟是多少？
    

以下部分的结构是介绍 Ftrace，展示它的一些分析器和跟踪器，然后展示使用它们的前端。这些部分是：

*   14.1：能力概述
    
*   14.2: 跟踪 (/sys)
    
*   分析器：
    
    *   14.3: FTrace 函数分析器
        
    *   14.10：Ftrace 历史触发器
        
*   追踪器：
    
    *   14.4: ftrace函数跟踪
        
    *   14.5: 跟踪点
        
    *   14.6: k探针
        
    *   14.7: 升级
        
    *   14.8: ftrace 函数图
        
    *   14.9: ftrace hwlat
        
*   前端：
    
    *   14.11: 跟踪命令
        
    *   14.12: 性能 ftrace
        
    *   14.13: 性能工具
        
*   14.14：Ftrace 文档
    
*   14.15: 参考文献
    

Ftrace hist 触发器是一个高级主题，需要首先介绍分析器和跟踪器，因此它位于本章后面。 kprobes 和 uprobes 部分还包括基本的分析功能。

图 14.1 是 Ftrace 及其前端的概述，其中箭头显示了从事件到输出类型的路径。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/14fig01.jpg)

图 14.1 FTrace 分析器、跟踪器和前端

这些将在以下各节中进行解释。

### 14.1 能力概述

虽然 perf(1) 使用子命令来实现不同的功能，但 Ftrace 具有分析器和跟踪器。分析器提供统计摘要，例如计数和直方图，跟踪器提供每个事件的详细信息。

作为 Ftrace 的示例，以下 funcgraph(8) 工具使用 Ftrace 跟踪器来显示 vfs\_read() 内核函数的子调用：

点击这里查看代码图片

\# **funcgraph vfs\_read**
Tracing "vfs\_read"... Ctrl-C to end.
 1)               |  vfs\_read() {
 1)               |    rw\_verify\_area() {
 1)               |      security\_file\_permission() {
 1)               |        apparmor\_file\_permission() {
 1)               |          common\_file\_perm() {
 1)   0.763 us    |            aa\_file\_perm();
 1)   2.209 us    |          }
 1)   3.329 us    |        }
 1)   0.571 us    |        \_\_fsnotify\_parent();
 1)   0.612 us    |        fsnotify();
 1)   7.019 us    |      }
 1)   8.416 us    |    }
 1)               |    \_\_vfs\_read() {
 1)               |      new\_sync\_read() {
 1)               |        ext4\_file\_read\_iter() {
\[...\]

输出显示 vfs\_read() 调用了 rw\_verify\_area()，rw\_verify\_area() 调用了 security\_file\_permission() 等等。第二列显示每个函数的持续时间（“us”是微秒），以便您可以进行性能分析，识别导致父函数缓慢的子函数。这种特殊的 Ftrace 功能称为函数图跟踪（在第 14.8 节 Ftrace function\_graph 中介绍）。

表 14.1 和 14.2 列出了来自最新 Linux 版本 (5.2) 的 Ftrace 分析器和跟踪器，以及 Linux 事件跟踪器：tracepoints、kprobes 和 uprobes。这些事件跟踪器与 Ftrace 类似，共享相似的配置和输出接口，并且包含在本章中。表 14.2 中显示的等宽跟踪器名称是 Ftrace 跟踪器，也是用于配置它们的命令行关键字。

表 14.1 FTrace 分析器

分析器

描述

部分

功能

核函数统计

[[ch14.xhtml#ch14lev3|14.3]]

kprobe配置文件

启用 kprobe 计数

[[ch14.xhtml#ch14lev6sec5|14.6.5]]

取消探测配置文件

启用 uprobe 计数

[[ch14.xhtml#ch14lev7sec4|14.7.4]]

历史触发器

事件的自定义直方图

[[ch14.xhtml#ch14lev10|14.10]]

表 14.2 Ftrace 和事件跟踪器

示踪剂

描述

部分

`function`

内核函数调用跟踪器

[[ch14.xhtml#ch14lev4|14.4]]

跟踪点

内核静态检测（事件跟踪器）

[[ch14.xhtml#ch14lev5|14.5]]

k探针

内核动态检测（事件跟踪器）

[[ch14.xhtml#ch14lev6|14.6]]

乌鲁布斯

用户级动态检测（事件跟踪器）

[[ch14.xhtml#ch14lev7|14.7]]

`function_graph`

使用子调用的层次图进行内核函数调用跟踪。

[[ch14.xhtml#ch14lev8|14.8]]

`wakeup`

测量最大 CPU 调度程序延迟

\-

`wakeup_rt`

测量实时 (RT) 任务的最大 CPU 调度程序延迟

\-

`irqsoff`

通过代码位置和延迟跟踪事件的 IRQ（中断禁用延迟）[^1]

\-

`preemptoff`

跟踪抢占禁用事件的代码位置和延迟

\-

`preemptirqsoff`

结合了 irqsoff 和 preemptoff 的跟踪器

\-

`blk`

块 I/O 跟踪器（由 blktrace(8) 使用）。

\-

`hwlat`

硬件延迟跟踪器：可以检测导致延迟的外部扰动

[[ch14.xhtml#ch14lev9|14.9]]

`mmiotrace`

跟踪模块对硬件的调用

\-

`nop`

禁用其他跟踪器的特殊跟踪器

\-

[^1]此（以及 preemptoff、preemptirqsoff）需要启用 CONFIG\_PREEMPTIRQ\_EVENTS。

您可以使用以下命令列出您的内核版本上可用的 Ftrace 跟踪器：

点击这里查看代码图片

\# **cat /sys/kernel/debug/tracing/available\_tracers**
hwlat blk mmiotrace function\_graph wakeup\_dl wakeup\_rt wakeup function nop

这是使用挂载在/sys下的tracefs接口，下一节将介绍该接口。后续部分介绍分析器、跟踪器以及使用它们的工具。

如果您想直接跳到基于 Ftrace 的工具，请查看第 14.13 节，perf-tools，其中包括前面显示的 funcgraph(8)。

未来的内核版本可能会向 Ftrace 添加更多分析器和跟踪器：查看 Linux 源中的 Documentation/trace/ftrace.rst \[Rostedt 08\] 下的 Ftrace 文档。

### 14.2 跟踪 (/sys)

使用Ftrace功能的接口是tracefs文件系统。它应该安装在 /sys/kernel/tracing 上；例如，通过使用：

点击这里查看代码图片

**mount -t tracefs tracefs /sys/kernel/tracing**

Ftrace最初是debugfs文件系统的一部分，直到它被分割成自己的tracefs。当debugfs被挂载时，它仍然通过挂载tracefs作为跟踪子目录来保留原始目录结构。您可以使用以下命令列出 debugfs 和 tracefs 挂载点：

点击这里查看代码图片

\# **mount -t debugfs,tracefs**
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
tracefs on /sys/kernel/debug/tracing type tracefs (rw,relatime)

此输出来自 Ubuntu 19.10，它显示 Tracefs 已安装在 /sys/kernel/debug/tracing 中。以下各节中的示例使用此位置，因为它仍在广泛使用，但将来它应该更改为 /sys/kernel/tracing。

请注意，如果 tracefs 无法挂载，一个可能的原因是您的内核是在没有 Ftrace 配置选项（CONFIG\_FTRACE 等）的情况下构建的。

#### 14.2.1 Tracefs 内容

一旦安装了tracefs，您应该能够在跟踪目录中看到控制和输出文件：

点击这里查看代码图片

\# **ls -F /sys/kernel/debug/tracing**
available\_events            max\_graph\_depth      stack\_trace\_filter
available\_filter\_functions  options/             synthetic\_events
available\_tracers           per\_cpu/             timestamp\_mode
buffer\_percent              printk\_formats       trace
buffer\_size\_kb              README               trace\_clock
buffer\_total\_size\_kb        saved\_cmdlines       trace\_marker
current\_tracer              saved\_cmdlines\_size  trace\_marker\_raw
dynamic\_events              saved\_tgids          trace\_options
dyn\_ftrace\_total\_info       set\_event            trace\_pipe
enabled\_functions           set\_event\_pid        trace\_stat/
error\_log                   set\_ftrace\_filter    tracing\_cpumask
events/                     set\_ftrace\_notrace   tracing\_max\_latency
free\_buffer                 set\_ftrace\_pid       tracing\_on
function\_profile\_enabled    set\_graph\_function   tracing\_thresh
hwlat\_detector/             set\_graph\_notrace    uprobe\_events
instances/                  snapshot             uprobe\_profile
kprobe\_events               stack\_max\_size
kprobe\_profile              stack\_trace

其中许多的名称都很直观。关键文件和目录包括表 14.3 中列出的内容。

表 14.3 Tracefs 密钥文件

文件

使用权

描述

`available_tracers`

读

列出可用的跟踪器（参见表 14.2）

`current_tracer`

读/写

显示当前启用的跟踪器

`function_profile_enabled`

读/写

启用函数分析器

`available_filter_functions`

读

列出要跟踪的可用函数

`set_ftrace_filter`

读/写

选择要跟踪的函数

`tracing_on`

读/写

用于启用/禁用输出环形缓冲区的开关

`trace`

读/写

跟踪器的输出（环形缓冲区）

`trace_pipe`

读

示踪剂的输出；此版本使用跟踪和块进行输入

`trace_options`

读/写

自定义跟踪缓冲区输出的选项

`trace_stat`（目录）

读/写

函数分析器的输出

`kprobe_events`

读/写

启用 kprobe 配置

`uprobe_events`

读/写

启用 uprobe 配置

`events`（目录）

读/写

事件跟踪器控制文件：tracepoints、kprobes、uprobes

`instances`（目录）

读/写

并发用户的 FTrace 实例

此 /sys 接口记录在 Linux 源代码中的 Documentation/trace/ftrace.rst \[Rostedt 08\] 下。它可以直接从 shell 或前端和库使用。例如，要查看当前是否有任何 Ftrace 跟踪器正在使用，您可以 cat(1) current\_tracer 文件：

点击这里查看代码图片

\# **cat /sys/kernel/debug/tracing/current\_tracer**
nop

输出显示nop（无操作），这意味着当前没有使用任何跟踪器。要启用跟踪器，请将其名称写入此文件。例如，要启用 blk 跟踪器：

点击这里查看代码图片

\# **echo blk > /sys/kernel/debug/tracing/current\_tracer**

其他 Ftrace 控制和输出文件也可以通过 echo(1) 和 cat(1) 使用。这意味着 Ftrace 的使用依赖几乎为零（只需要一个 shell[^2]）。

[^2]echo(1) 是 shell 内置函数，cat(1) 可以近似为：函数 shellcat `{ (while read line; do echo "$line"; done) < $1; }`。或者，busybox 可用于包含 shell、cat(1) 和其他基础知识。

Steven Rostedt 在开发实时补丁集时构建了 Ftrace 供自己使用，最初它不支持并发用户。例如，current\_tracer 文件一次只能设置一个跟踪器。后来添加了并发用户支持，以可以在“instances”目录中创建的实例的形式。每个实例都有自己的 current\_tracer 和输出文件，以便可以独立执行跟踪。

以下部分（14.3 至 14.10）显示更多 /sys 接口示例；然后后面的部分（14.11 到 14.13）显示了基于它构建的前端：trace-cmd、perf(1) `ftrace` 子命令和 perf-tools。

### 14.3 FTrace 函数分析器

函数分析器提供有关内核函数调用的统计信息，适合探索哪些内核函数正在使用并识别哪些是最慢的。我经常使用函数分析器作为了解给定工作负载的内核代码执行的起点，特别是因为它高效且开销相对较低。使用它，我可以识别要使用更昂贵的每个事件跟踪进行分析的函数。它需要 CONFIG\_FUNCTION\_PROFILER=y 内核选项。

函数分析器通过在每个内核函数开始时使用编译的分析调用来工作。这种方法基于编译器分析器的工作方式，例如 gcc(1) 的 `-pg` 选项，该选项插入 mcount() 调用以与 gprof(1) 一起使用。从 gcc(1) 4.6 版本开始，这个 mcount() 调用现在是 \_\_fentry\_\_()。添加对每个内核函数的调用听起来应该会花费大量开销，这对于可能很少使用的东西来说是一个问题，但开销问题已经解决：当不使用时，这些调用通常会被快速 nop 指令替换，并且仅在需要时切换到 \_\_fentry\_\_() 调用 \[Gregg 19f\]。

下面演示了使用/sys中的tracefs接口的函数分析器。作为参考，以下显示了函数分析器的原始未启用状态：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **cat set\_ftrace\_filter**
#### all functions enabled ####
# **cat function\_profile\_enabled**
0

现在（来自同一目录）这些命令使用函数分析器来计算所有以“tcp”开头的内核调用大约 10 秒：

点击这里查看代码图片

\# **echo 'tcp\*' > set\_ftrace\_filter**
# **echo 1 > function\_profile\_enabled**
# **sleep 10**
# **echo 0 > function\_profile\_enabled**
# **echo > set\_ftrace\_filter**

sleep(1) 命令用于设置配置文件的（粗略）持续时间。之后的命令禁用功能分析并重置过滤器。提示：请务必使用“`0 >`”而不是“`0>`”——它们不一样；后者是文件描述符 0 的重定向。同样避免“1`>`”，因为它是文件描述符 1 的重定向。

现在可以从trace\_stat目录中读取配置文件统计信息，该目录将它们保存在每个CPU的“函数”文件中。这是一个 2-CPU 系统。使用 head(1) 仅显示每个文件的前十行：

点击这里查看代码图片

\# **head trace\_stat/function\***
==> trace\_stat/function0 <==
  Function                       Hit    Time            Avg             s^2
  --------                       ---    ----            ---             ---
  tcp\_sendmsg                 955912    2788479 us      2.917 us        3734541 us
  tcp\_sendmsg\_locked          955912    2248025 us      2.351 us        2600545 us
  tcp\_push                    955912    852421.5 us     0.891 us        1057342 us
  tcp\_write\_xmit              926777    674611.1 us     0.727 us        1386620 us
  tcp\_send\_mss                955912    504021.1 us     0.527 us        95650.41 us
  tcp\_current\_mss             964399    317931.5 us     0.329 us        136101.4 us
  tcp\_poll                    966848    216701.2 us     0.224 us        201483.9 us
  tcp\_release\_cb              956155    102312.4 us     0.107 us        188001.9 us

==> trace\_stat/function1 <==
  Function                       Hit    Time            Avg             s^2
  --------                       ---    ----            ---             ---
  tcp\_sendmsg                 317935    936055.4 us     2.944 us        13488147 us
  tcp\_sendmsg\_locked          317935    770290.2 us     2.422 us        8886817 us
  tcp\_write\_xmit              348064    423766.6 us     1.217 us        226639782 us
  tcp\_push                    317935    310040.7 us     0.975 us        4150989 us
  tcp\_tasklet\_func             38109    189797.2 us     4.980 us        2239985 us
  tcp\_tsq\_handler              38109    180516.6 us     4.736 us        2239552 us
  tcp\_tsq\_write.part.0         29977    173955.7 us     5.802 us        1037352 us
  tcp\_send\_mss                317935    165881.9 us     0.521 us        352309.0 us

这些列显示函数名称 (`Function`)、调用次数 (`Hit`)、函数总时间 (`Time`)、平均函数时间 (`Avg`)和标准差 (`s^2`)。输出显示 tcp\_sendmsg() 函数在两个 CPU 上最频繁；它在 CPU0 上被调用超过 955k 次，在 CPU1 上被调用超过 317k 次。其平均持续时间为 2.9 微秒。

在分析过程中，少量开销会添加到分析函数中。如果 set\_ftrace\_filter 留空，则会分析所有内核函数（正如我们之前看到的初始状态所警告的那样：“所有函数已启用”）。使用探查器时请记住这一点，并尝试使用函数过滤器来限制开销。

稍后介绍的 Ftrace 前端可以自动执行这些步骤，并且可以将每个 CPU 的输出合并到系统范围的摘要中。

### 14.4 ftrace函数跟踪

函数跟踪器打印内核函数调用的每个事件详细信息，并使用上一节中描述的函数分析工具。这可以显示各种函数的顺序、基于时间戳的模式以及可能负责的 CPU 上进程名称和 PID。函数跟踪的开销高于函数分析，因此跟踪最适合相对不频繁的函数（每秒调用次数少于 1,000 次）。您可以使用上一节中的函数分析来在跟踪函数之前找出函数的速率。

函数跟踪涉及的关键tracefs文件如图14.2所示。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/14fig02.jpg)

图14.2 ftrace函数跟踪tracefs文件

最终跟踪输出从 `trace` 或 `trace_pipe` 文件读取，如以下部分所述。这两个接口也都有清除输出缓冲区的方法（因此箭头返回到缓冲区）。

#### 14.4.1 使用跟踪

下面演示了使用 `trace` 输出文件进行函数跟踪。作为参考，下面显示了函数跟踪器的原始未启用状态：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **cat set\_ftrace\_filter**
#### all functions enabled ####
# **cat current\_tracer**
nop

目前没有使用其他示踪剂。

对于此示例，将跟踪所有以“sleep”结尾的内核函数，并且事件最终保存到 /tmp/out.trace01.txt 文件中。虚拟 sleep(1) 命令用于收集至少 10 秒的跟踪。此命令序列通过禁用函数跟踪器并使系统恢复正常来完成：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **echo 1 > tracing\_on**
# **echo '\*sleep' > set\_ftrace\_filter**
# **echo function > current\_tracer**
# **sleep 10**
# **cat trace > /tmp/out.trace01.txt**
# **echo nop > current\_tracer**
# **echo > set\_ftrace\_filter**

设置tracing\_on可能是一个不必要的步骤（在我的Ubuntu系统上，默认设置为1）。我已将其包含在内，以防您的系统上未设置它。

当我们跟踪“睡眠”函数调用时，在跟踪输出中捕获了虚拟 sleep(1) 命令：

点击这里查看代码图片

\# **more /tmp/out.trace01.txt**
# tracer: function
#
# entries-in-buffer/entries-written: 57/57   #P:2
#
#                              \_-----=> irqs-off
#                             / \_----=> need-resched
#                            | / \_---=> hardirq/softirq
#                            || / \_--=> preempt-depth
#                            ||| /     delay
#           TASK-PID   CPU#  ||||    TIMESTAMP  FUNCTION
#              | |       |   ||||       |         |
      multipathd-348   \[001\] .... 332762.532877: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
      multipathd-348   \[001\] .... 332762.532879: hrtimer\_nanosleep <-
\_\_x64\_sys\_nanosleep
      multipathd-348   \[001\] .... 332762.532880: do\_nanosleep <-hrtimer\_nanosleep
           sleep-4203  \[001\] .... 332762.722497: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
           sleep-4203  \[001\] .... 332762.722498: hrtimer\_nanosleep <-
\_\_x64\_sys\_nanosleep
           sleep-4203  \[001\] .... 332762.722498: do\_nanosleep <-hrtimer\_nanosleep
      multipathd-348   \[001\] .... 332763.532966: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
\[...\]

输出包括字段标题和跟踪元数据。此示例显示名为 `multipathd` 且进程 ID 为 348 的进程调用 sleep 函数以及 sleep(1) 命令。最后的字段显示当前函数和调用它的父函数。例如，对于第一行，函数是 \_\_x64\_sys\_nanosleep()，并由 do\_syscall\_64() 调用。

`trace` 文件是跟踪事件缓冲区的接口。读取它显示缓冲区内容；您可以通过写入换行符来清除内容：

\# **\> trace**

当 `current_tracer` 设置回 `nop` 时，跟踪缓冲区也会被清除，就像我在禁用跟踪的示例步骤中所做的那样。当使用trace\_pipe时它也会被清除。

#### 14.4.2 使用trace\_pipe

`trace_pipe` 文件是用于读取跟踪缓冲区的不同接口。从此文件读取会返回无尽的事件流。它还消耗事件，因此在读取它们后，一旦它们不再位于跟踪缓冲区中。

例如，使用 `trace_pipe` 实时观看睡眠事件：

点击这里查看代码图片

\# **echo '\*sleep' > set\_ftrace\_filter**
# **echo function > current\_tracer**
# **cat trace\_pipe**
      multipathd-348   \[001\] .... 332624.519190: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
      multipathd-348   \[001\] .... 332624.519192: hrtimer\_nanosleep <-
\_\_x64\_sys\_nanosleep
      multipathd-348   \[001\] .... 332624.519192: do\_nanosleep <-hrtimer\_nanosleep
      multipathd-348   \[001\] .... 332625.519272: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
      multipathd-348   \[001\] .... 332625.519274: hrtimer\_nanosleep <-
\_\_x64\_sys\_nanosleep
      multipathd-348   \[001\] .... 332625.519275: do\_nanosleep <-hrtimer\_nanosleep
            cron-504   \[001\] .... 332625.560150: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
            cron-504   \[001\] .... 332625.560152: hrtimer\_nanosleep <-
\_\_x64\_sys\_nanosleep
            cron-504   \[001\] .... 332625.560152: do\_nanosleep <-hrtimer\_nanosleep
^C
# **echo nop > current\_tracer**
# **echo > set\_ftrace\_filter**

输出显示了 `multipathd` 和 `cron` 进程的睡眠次数。这些字段与前面显示的跟踪文件输出相同，但这次没有列标题。

`trace_pipe` 文件对于观察低频事件很方便，但对于高频事件，您需要将它们捕获到文件中，以便稍后使用前面显示的 `trace` 文件进行分析。

#### 14.4.3 选项

Ftrace 提供了用于自定义跟踪输出的选项，可以通过trace\_options 文件或选项目录进行控制。例如（来自同一目录）禁用标志列（在之前的输出中为“...”）：

点击这里查看代码图片

\# **echo 0 > options/irq-info**
# **cat trace**
# tracer: function
#
# entries-in-buffer/entries-written: 3300/3300   #P:2
#
#           TASK-PID     CPU#   TIMESTAMP  FUNCTION
#              | |         |       |         |
      multipathd-348   \[001\]  332762.532877: \_\_x64\_sys\_nanosleep <-do\_syscall\_64
      multipathd-348   \[001\]  332762.532879: hrtimer\_nanosleep <-\_\_x64\_sys\_nanosleep
      multipathd-348   \[001\]  332762.532880: do\_nanosleep <-hrtimer\_nanosleep
\[...\]

现在输出中不存在标志文件。您可以使用以下方法将其重新设置：

\# **echo 1 > options/irq-info**

还有更多选项，您可以从选项目录中列出它们；他们有一些直观的名字。

点击这里查看代码图片

\# **ls options/**
annotate          funcgraph-abstime   hex              stacktrace
bin               funcgraph-cpu       irq-info         sym-addr
blk\_cgname        funcgraph-duration  latency-format   sym-offset
blk\_cgroup        funcgraph-irqs      markers          sym-userobj
blk\_classic       funcgraph-overhead  overwrite        test\_nop\_accept
block             funcgraph-overrun   print-parent     test\_nop\_refuse
context-info      funcgraph-proc      printk-msg-only  trace\_printk
disable\_on\_free   funcgraph-tail      raw              userstacktrace
display-graph     function-fork       record-cmd       verbose
event-fork        function-trace      record-tgid
func\_stack\_trace  graph-time          sleep-time

这些选项包括 `stacktrace` 和 `userstacktrace`，它们会将内核和用户堆栈跟踪附加到输出：这对于理解调用函数的原因很有用。所有这些选项都记录在 Linux 源代码 \[Rostedt 08\] 的 Ftrace 文档中。

### 14.5 跟踪点

跟踪点是内核静态检测，在第 4 章“可观察性工具”、第 4.3.5 节“跟踪点”中介绍。从技术上讲，跟踪点只是放置在内核源代码中的跟踪函数；它们是从定义和格式化其参数的跟踪事件接口中使用的。跟踪事件在tracefs中可见，并与Ftrace共享输出和控制文件。

例如，以下代码启用 block:block\_rq\_issue 跟踪点并实时监视事件。此示例通过禁用跟踪点来完成：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **echo 1 > events/block/block\_rq\_issue/enable**
# **cat trace\_pipe**
            sync-4844  \[001\] .... 343996.918805: block\_rq\_issue: 259,0 WS 4096 ()
2048 + 8 \[sync\]
            sync-4844  \[001\] .... 343996.918808: block\_rq\_issue: 259,0 WSM 4096 ()
10560 + 8 \[sync\]
            sync-4844  \[001\] .... 343996.918809: block\_rq\_issue: 259,0 WSM 4096 ()
38424 + 8 \[sync\]
            sync-4844  \[001\] .... 343996.918809: block\_rq\_issue: 259,0 WSM 4096 ()
4196384 + 8 \[sync\]
            sync-4844  \[001\] .... 343996.918810: block\_rq\_issue: 259,0 WSM 4096 ()
4462592 + 8 \[sync\]
^C
# **echo 0 > events/block/block\_rq\_issue/enable**

前五列与 4.6.4 中所示相同，分别是：进程名称“-”PID、CPU ID、标志、时间戳（秒）和事件名称。其余部分是跟踪点的格式字符串，如第 4.3.5 节所述。

在此示例中可以看出，跟踪点在事件下的目录结构中具有控制文件。每个跟踪系统都有一个目录（例如“block”），并且在每个事件的这些子目录中（例如“block\_rq\_issue”）。列出该目录：

点击这里查看代码图片

**\# ls events/block/block\_rq\_issue/**
enable  filter  format  hist  id  trigger

这些控制文件记录在 Linux 源代码中的 Documentation/trace/events.rst \[Ts’o 20\] 下。在此示例中，`enable` 文件用于打开和关闭跟踪点。其他文件提供过滤和触发功能。

#### 14.5.1 过滤器

可以包含过滤器以仅在满足布尔表达式时记录事件。它有一个受限制的语法：

field operator value

该字段来自第 4.3.5 节中描述的格式文件，标题为 Tracepoints Arguments 和 Format String（这些字段也以前面描述的格式字符串打印）。数字运算符是以下之一：==、!=、<、<=、>、>=、&；对于字符串：==、!=、~。 “~”运算符执行 shell 全局样式匹配，使用通配符：\*、?、\[\]。这些布尔表达式可以用括号分组并使用：&&、|| 进行组合。

作为示例，以下内容在已启用的 block:block\_rq\_insert 跟踪点上设置过滤器，以仅跟踪字节字段大于 64 KB 的事件：

点击这里查看代码图片

\# **echo 'bytes > 65536' > events/block/block\_rq\_insert/filter**
# **cat trace\_pipe**
    kworker/u4:1-7173  \[000\] .... 378115.779394: block\_rq\_insert: 259,0 W 262144 ()
5920256 + 512 \[kworker/u4:1\]
    kworker/u4:1-7173  \[000\] .... 378115.784654: block\_rq\_insert: 259,0 W 262144 ()
5924336 + 512 \[kworker/u4:1\]
    kworker/u4:1-7173  \[000\] .... 378115.789136: block\_rq\_insert: 259,0 W 262144 ()
5928432 + 512 \[kworker/u4:1\]
^C

输出现在仅包含更大的 I/O。

点击这里查看代码图片

\# **echo 0 > events/block/block\_rq\_insert/filter**

此 `echo 0` 会重置过滤器。

#### 14.5.2 触发

当事件触发时，触发器会运行额外的跟踪命令。该命令可能是启用或禁用其他跟踪、打印堆栈跟踪或拍摄跟踪缓冲区的快照。当前未设置触发器时，可以从触发器文件中列出可用的触发器命令。例如：

点击这里查看代码图片

\# **cat events/block/block\_rq\_issue/trigger**
# Available triggers:
# traceon traceoff snapshot stacktrace enable\_event disable\_event enable\_hist
disable\_hist hist

触发器的一个用例是当您希望查看导致错误情况的事件时：可以在错误情况下放置触发器，以禁用跟踪 (`traceoff`)，以便跟踪缓冲区仅包含先前的事件，或拍摄快照 (`snapshot`) 来保存它。

触发器可以通过使用 `if` 关键字与过滤器结合使用，如上一节所示。这对于匹配错误条件或有趣的事件可能是必要的。例如，当大于 64 KB 的块 I/O 排队时停止记录事件：

点击这里查看代码图片

\# **echo 'traceoff if bytes > 65536' > events/block/block\_rq\_insert/trigger**

可以使用 hist 触发器执行更复杂的操作，第 14.10 节“Ftrace Hist 触发器”中介绍了这些操作。

### 14.6 k探针

kprobes 是内核动态检测，在第 4 章“可观测性工具”的 4.3.6 节“kprobes”中介绍。 kprobes 创建 kprobe 事件供跟踪器使用，它与 Ftrace 共享 Tracefs 输出和控制文件。 kprobes 类似于 14.4 节中介绍的 Ftrace 函数跟踪器，因为它们跟踪内核函数。然而，kprobes 可以进一步定制，可以放置在函数偏移量（单独的指令）上，并且可以报告函数参数和返回值。

本节介绍 kprobe 事件跟踪和 Ftrace kprobe 分析器。

#### 14.6.1 事件跟踪

作为示例，以下使用 kprobes 来检测 do\_nanosleep() 内核函数：

点击这里查看代码图片

\# **echo 'p:brendan do\_nanosleep' >> kprobe\_events**
# **echo 1 > events/kprobes/brendan/enable**
# **cat trace\_pipe**
      multipathd-348   \[001\] .... 345995.823380: brendan: (do\_nanosleep+0x0/0x170)
      multipathd-348   \[001\] .... 345996.823473: brendan: (do\_nanosleep+0x0/0x170)
      multipathd-348   \[001\] .... 345997.823558: brendan: (do\_nanosleep+0x0/0x170)
^C
# **echo 0 > events/kprobes/brendan/enable**
**\# echo '-:brendan' >> kprobe\_events**

通过向 kprobe\_events 附加特殊语法来创建和删除 kprobe。创建后，它与跟踪点一起出现在事件目录中，并且可以以类似的方式使用。

kprobe 语法在 Documentation/trace/kprobetrace.rst \[Hiramatsu 20\] 下的内核源代码中有完整解释。 kprobes 能够跟踪内核函数的进入和返回以及函数偏移量。概要是：

点击这里查看代码图片

  p\[:\[GRP/\]EVENT\] \[MOD:\]SYM\[+offs\]|MEMADDR \[FETCHARGS\]  : Set a probe
  r\[MAXACTIVE\]\[:\[GRP/\]EVENT\] \[MOD:\]SYM\[+0\] \[FETCHARGS\]  : Set a return probe
  -:\[GRP/\]EVENT                                         : Clear a probe

在我的示例中，字符串“`p:brendan do_nanosleep`”为内核符号 do\_nanosleep() 创建名称为“brendan”的探针 (p:)。字符串“`-:brendan`”删除名为“brendan”的探针。

事实证明，自定义名称对于区分 kprobes 的不同用户非常有用。 BCC 跟踪器（在第 15 章 BPF、第 15.1 节 BCC 中介绍）使用包含跟踪函数、字符串“bcc”和 BCC PID 的名称。例如：

点击这里查看代码图片

\# **cat /sys/kernel/debug/tracing/kprobe\_events**
p:kprobes/**p\_blk\_account\_io\_start\_bcc\_19454** blk\_account\_io\_start
p:kprobes/**p\_blk\_mq\_start\_request\_bcc\_19454** blk\_mq\_start\_request

请注意，在较新的内核上，BCC 已切换为使用基于 perf\_event\_open(2) 的接口来使用 kprobe 而不是 kprobe\_events 文件（并且使用 perf\_event\_open(2) 启用的事件不会出现在 kprobe\_events 中）。

#### 14.6.2 参数

与函数跟踪（第 14.4 节，Ftrace 函数跟踪）不同，kprobes 可以检查函数参数和返回值。作为示例，以下是之前跟踪的 do\_nanosleep() 函数的声明，来自 kernel/time/hrtimer.c，其中突出显示了参数变量类型：

点击这里查看代码图片

static int \_\_sched do\_nanosleep(struct **hrtimer\_sleeper** \*t, enum **hrtimer\_mode** mode)
{
\[...\]

跟踪 Intel x86\_64 系统上的前两个参数并将它们打印为十六进制（默认值）：

点击这里查看代码图片

\# **echo 'p:brendan do\_nanosleep hrtimer\_sleeper=$arg1 hrtimer\_mode=$arg2' >>
kprobe\_events**
# **echo 1 > events/kprobes/brendan/enable**
# **cat trace\_pipe**
      multipathd-348   \[001\] .... 349138.128610: brendan: (do\_nanosleep+0x0/0x170)
**hrtimer\_sleeper=0xffffaa6a4030be80 hrtimer\_mode=0x1**
      multipathd-348   \[001\] .... 349139.128695: brendan: (do\_nanosleep+0x0/0x170)
**hrtimer\_sleeper=0xffffaa6a4030be80 hrtimer\_mode=0x1**
      multipathd-348   \[001\] .... 349140.128785: brendan: (do\_nanosleep+0x0/0x170)
**hrtimer\_sleeper=0xffffaa6a4030be80 hrtimer\_mode=0x1**
^C
# **echo 0 > events/kprobes/brendan/enable**
# **echo '-:brendan' >> kprobe\_events**

在第一行的事件描述中添加了额外的语法：例如，字符串“`hrtimer_sleeper=$arg1`”跟踪函数的第一个参数并使用自定义名称“hrtimer\_sleeper”。这已在输出中突出显示。

Linux 4.20 中添加了以 $arg1、$arg2 等形式访问函数参数的功能。之前的 Linux 版本需要使用寄存器名称。[^3] 以下是使用寄存器名称的等效 kprobe 定义：

[^3]对于尚未添加别名的处理器架构来说，这可能也是必要的。

点击这里查看代码图片

\# **echo 'p:brendan do\_nanosleep hrtimer\_sleeper=%di hrtimer\_mode=%si' >> kprobe\_events**

要使用寄存器名称，您需要了解处理器类型和所使用的函数调用约定。 x86\_64 使用 AMD64 ABI \[Matz 13\]，因此前两个参数在寄存器 rdi 和 rsi 中可用。[^4] perf(1) 也使用此语法，我在中提供了一个更复杂的示例第 13 章，perf，第 13.7.2 节，uprobes，取消引用字符串指针。

[^4]syscall(2) 手册页总结了不同处理器的调用约定。摘录于第 14.13.4 节，perf-tools One-Liners。

#### 14.6.3 返回值

返回值的特殊别名 `$retval` 可与 kretprobes 一起使用。以下示例使用它来显示 do\_nanosleep() 的返回值：

点击这里查看代码图片

\# **echo 'r:brendan do\_nanosleep ret=$retval' >> kprobe\_events**
# **echo 1 > events/kprobes/brendan/enable**
# **cat trace\_pipe**
      multipathd-348   \[001\] d... 349782.180370: brendan:
(hrtimer\_nanosleep+0xce/0x1e0 <- do\_nanosleep) **ret=0x0**
      multipathd-348   \[001\] d... 349783.180443: brendan:
(hrtimer\_nanosleep+0xce/0x1e0 <- do\_nanosleep) **ret=0x0**
      multipathd-348   \[001\] d... 349784.180530: brendan:
(hrtimer\_nanosleep+0xce/0x1e0 <- do\_nanosleep) **ret=0x0**
^C
# **echo 0 > events/kprobes/brendan/enable**
# **echo '-:brendan' >> kprobe\_events**

此输出显示，在跟踪时，do\_nanosleep() 的返回值始终为“`0`”（成功）。

#### 14.6.4 过滤器和触发器

过滤器和触发器可以在 events/kprobes/... 目录中使用，就像它们与跟踪点一样（参见第 14.5 节，跟踪点）。以下是带有参数的 do\_nanosleep() 上早期 kprobe 的格式文件（来自第 14.6.2 节，参数）：

点击这里查看代码图片

\# **cat events/kprobes/brendan/format**
name: brendan
ID: 2024
format:
        field:unsigned short common\_type;  offset:0;  size:2;    signed:0;
        field:unsigned char common\_flags;  offset:2;  size:1;    signed:0;
        field:unsigned char common\_preempt\_count;   offset:3;  size:1;  signed:0;
        field:int common\_pid;    offset:4; size:4;    signed:1;

        field:unsigned long \_\_probe\_ip;    offset:8;   size:8;   signed:0;
        field:u64 **hrtimer\_sleeper**;     offset:16;  size:8;    signed:0;
        field:u64 **hrtimer\_mode**;  offset:24;     size:8;     signed:0;
print fmt: "(%lx) hrtimer\_sleeper=0x%Lx hrtimer\_mode=0x%Lx", REC->\_\_probe\_ip, REC-
>hrtimer\_sleeper, REC->hrtimer\_mode

请注意，我的自定义 hrtimer\_sleeper 和 hrtimer\_mode 变量名称作为可与过滤器一起使用的字段可见。例如：

点击这里查看代码图片

\# **echo 'hrtimer\_mode != 1' > events/kprobes/brendan/filter**

这只会跟踪 hrtimer\_mode 不等于 1 的 do\_nanosleep() 调用。

#### 14.6.5 kprobe 分析

当启用 kprobes 时，Ftrace 会对它们的事件进行计数。这些计数可以打印在 kprobe\_profile 文件中。例如：

点击这里查看代码图片

\# **cat /sys/kernel/debug/tracing/kprobe\_profile**
  p\_blk\_account\_io\_start\_bcc\_19454                        1808               0
  p\_blk\_mq\_start\_request\_bcc\_19454                         677               0
  p\_blk\_account\_io\_completion\_bcc\_19454                    521              11
  p\_kbd\_event\_1\_bcc\_1119                                   632               0

这些列是：探针名称（可以通过打印 kprobe\_events 文件查看其定义）、命中计数和未命中计数（探针被命中，但随后遇到错误且未记录：它被错过了）。

虽然您已经可以使用函数分析器（第 14.3 节）获取函数计数，但我发现 kprobe 分析器对于检查监控软件使用的始终启用的 kprobes 非常有用，以防某些 kprobe 触发过于频繁并且应该禁用（如果可能） ）。

### 14.7 升级

uprobes 是用户级动态检测，在第 4 章“可观测性工具”的 4.3.7 节“uprobes”中介绍。 uprobes 创建 uprobe 事件供跟踪器使用，它与 Ftrace 共享 Tracefs 输出和控制文件。

本节介绍 uprobe 事件跟踪和 Ftrace uprobe 分析器。

#### 14.7.1 事件跟踪

对于 uprobes，控制文件是 uprobe\_events，其语法记录在 Linux 源文件 Documentation/trace/uprobetracer.rst \[Dronamraju 20\] 下。概要是：

点击这里查看代码图片

  p\[:\[GRP/\]EVENT\] PATH:OFFSET \[FETCHARGS\] : Set a uprobe
  r\[:\[GRP/\]EVENT\] PATH:OFFSET \[FETCHARGS\] : Set a return uprobe (uretprobe)
  -:\[GRP/\]EVENT                           : Clear uprobe or uretprobe event

现在，语法需要 uprobe 的路径和偏移量。内核没有用户空间软件的符号信息，因此必须使用用户空间工具确定该偏移量并将其提供给内核。

以下示例使用 uprobes 从 bash(1) shell 检测 readline() 函数，首先查找符号偏移量：

点击这里查看代码图片

\# **readelf -s /bin/bash | grep -w readline**
   882: 00000000000b61e0   153 FUNC    GLOBAL DEFAULT   14 readline
# **echo 'p:brendan /bin/bash:0xb61e0' >> uprobe\_events**
# **echo 1 > events/uprobes/brendan/enable**
# **cat trace\_pipe**
            bash-3970  \[000\] d... 347549.225818: brendan: (0x55d0857b71e0)
            bash-4802  \[000\] d... 347552.666943: brendan: (0x560bcc1821e0)
            bash-4802  \[000\] d... 347552.799480: brendan: (0x560bcc1821e0)
^C
# **echo 0 > events/uprobes/brendan/enable**
**\# echo '-:brendan' >> uprobe\_events**

警告：如果您错误地使用指令中间的符号偏移量，则会损坏目标进程（对于共享指令文本，所有共享它的进程！）。如果目标二进制文件已编译为具有地址空间布局随机化 (ASLR) 的位置无关可执行文件 (PIE)，则使用 readelf(1) 查找符号偏移量的示例技术可能不起作用。我根本不建议您使用此接口：切换到为您处理符号映射的更高级别跟踪器（例如，BCC 或 bpftrace）。

#### 14.7.2 参数和返回值

这些与第 14.6 节 kprobes 中演示的 kprobes 类似。可以通过在创建 uprobe 时指定它们来检查 uprobe 参数和返回值。语法位于 uprobetracer.rst \[Dronamraju 20\] 中。

#### 14.7.3 过滤器和触发器

过滤器和触发器可以在 events/uprobes/... 目录中使用，就像它们与 kprobes 一起使用一样（参见第 14.6 节，kprobes）。

#### 14.7.4 uprobe 分析

当启用 uprobes 时，Ftrace 会对其事件进行计数。这些计数可以打印在 uprobe\_profile 文件中。例如：

点击这里查看代码图片

\# **cat /sys/kernel/debug/tracing/uprobe\_profile**
  /bin/bash brendan                                                   11

这些列是：路径、探针名称（可以通过打印 uprobe\_events 文件查看其定义）和命中计数。

### 14.8 FTrace函数图

function\_graph 跟踪器打印函数的调用图，揭示代码流程。本章从 perf-tools 中的 funcgraph(8) 的示例开始。下图展示了Ftrace的tracefs界面。

作为参考，这里是函数图跟踪器的原始未启用状态：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
**\# cat set\_graph\_function**
#### all functions enabled ####
# **cat current\_tracer**
nop

目前没有使用其他示踪剂。

#### 14.8.1 图形追踪

下面使用 do\_nanosleep() 函数上的 function\_graph 跟踪器来显示其子函数调用：

点击这里查看代码图片

\# **echo do\_nanosleep > set\_graph\_function**
# **echo function\_graph > current\_tracer**
# **cat trace\_pipe**
 1)   2.731 us    |  get\_xsave\_addr();
 1)               |  do\_nanosleep() {
 1)               |    hrtimer\_start\_range\_ns() {
 1)               |      lock\_hrtimer\_base.isra.0() {
 1)   0.297 us    |        \_raw\_spin\_lock\_irqsave();
 1)   0.843 us    |      }
 1)   0.276 us    |      ktime\_get();
 1)   0.340 us    |      get\_nohz\_timer\_target();
 1)   0.474 us    |      enqueue\_hrtimer();
 1)   0.339 us    |      \_raw\_spin\_unlock\_irqrestore();
 1)   4.438 us    |    }
 1)               |    schedule() {
 1)               |      rcu\_note\_context\_switch() {
\[...\]
 5) $ 1000383 us  |  } /\* do\_nanosleep \*/
^C
# **echo nop > current\_tracer**
# **echo > set\_graph\_function**

输出显示了子调用和代码流：do\_nanosleep() 调用了 hrtimer\_start\_range\_ns()，后者调用了 lock\_hrtimer\_base.isra.0()，依此类推。左侧的列显示 CPU（在此输出中，主要是 CPU 1）和函数的持续时间，以便可以识别延迟。高延迟包含一个字符符号来帮助您引起注意，在此输出中，1000383 微秒（1.0 秒）的延迟旁边有一个“`”。角色是\[Rostedt 08\]：

*   `：大于 1 秒
    
*   `@`：大于 100 毫秒
    
*   `*`：大于 10 毫秒
    
*   `#`：大于 1 毫秒
    
*   `!`：大于 100 μs
    
*   `+`：大于 10 μs
    

这个例子故意没有设置函数过滤器（set\_ftrace\_filter），这样所有的子调用都可以看到。然而，这确实会花费一些开销，从而夸大了报告的持续时间。它通常对于定位高延迟的根源仍然有用，这可以使增加的开销相形见绌。当您希望给定函数的时间更准确时，可以使用函数过滤器来减少跟踪的函数。例如，仅跟踪 do\_nanosleep()：

点击这里查看代码图片

\# **echo do\_nanosleep > set\_ftrace\_filter**
# **cat trace\_pipe**
\[...\]
 7) $ 1000130 us  |  } /\* do\_nanosleep \*/
^C

我正在跟踪相同的工作负载 (`sleep 1`)。应用过滤器后， do\_nanosleep() 报告的持续时间已从 1000383 μs 下降到 1000130 μs（对于这些示例输出），因为它不再包括跟踪所有子函数的开销。

这些示例还使用trace\_pipe实时观察输出，但这很冗长，并且将跟踪文件重定向到输出文件更实用，正如我在第14.4节“Ftrace函数跟踪”中演示的那样。

#### 14.8.2 选项

可以使用选项来更改输出，这些选项可以在选项目录中列出：

点击这里查看代码图片

\# **ls options/funcgraph-\***
options/funcgraph-abstime   options/funcgraph-irqs      options/funcgraph-proc
options/funcgraph-cpu       options/funcgraph-overhead  options/funcgraph-tail
options/funcgraph-duration  options/funcgraph-overrun

它们调整输出，并可以包含或排除详细信息，例如 CPU ID (funcgraph-cpu)、进程名称 (funcgraph-proc)、函数持续时间 (funcgraph-duration) 和延迟标记 (funcgraph-overhead)。

### 14.9 ftrace hwlat

硬件延迟检测器 (hwlat) 是专用跟踪器的一个示例。它可以检测外部硬件事件何时扰乱 CPU 性能：这些事件对于内核和其他工具来说是不可见的。例如，系统管理中断 (SMI) 事件和虚拟机管理程序扰动（包括由嘈杂的邻居引起的扰动）。

这是通过运行代码循环作为禁用中断的实验来实现的，测量循环每次迭代运行所需的时间。该循环一次在一个 CPU 上执行，并在它们之间轮流执行。如果每个 CPU 的最慢循环迭代超过阈值（10 微秒，可以通过tracing\_thresh 文件进行配置），则会打印该最慢的循环迭代。

这是一个例子：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **echo hwlat > current\_tracer**
# **cat trace\_pipe**
           <...>-5820  \[001\] d... 354016.973699: #1     inner/outer(us): 2152/1933
ts:1578801212.559595228
           <...>-5820  \[000\] d... 354017.985568: #2     inner/outer(us):   19/26
ts:1578801213.571460991
           <...>-5820  \[001\] dn.. 354019.009489: #3     inner/outer(us): 1699/5894
ts:1578801214.595380588
           <...>-5820  \[000\] d... 354020.033575: #4     inner/outer(us):   43/49
ts:1578801215.619463259
           <...>-5820  \[001\] d... 354021.057566: #5     inner/outer(us):   18/45
ts:1578801216.643451721
           <...>-5820  \[000\] d... 354022.081503: #6     inner/outer(us):   18/38
ts:1578801217.667385514
^C
# **echo nop > current\_tracer**

其中许多字段已在前面的部分中进行了描述（请参见第 14.4 节，Ftrace 函数跟踪）。有趣的是时间戳后面：有一个序列号（#1，...），然后是“内部/外部（us）”数字，以及最终时间戳。内部/外部数字显示循环内部（内部）的循环计时以及包装到下一个循环迭代（外部）的代码逻辑。第一行显示迭代耗时 2,152 微秒（内部）和 1,933 微秒（外部）。这远远超过了 10 微秒的阈值，并且是由于外部扰动造成的。

hwlat 有可以配置的参数：循环运行一段称为宽度的时间，并在称为窗口的时间段内运行一个宽度实验。记录每个宽度期间长于阈值（10 微秒）的最慢迭代。这些参数可以通过 /sys/kernel/debug/tracing/hwlat\_detector 中的文件进行修改：宽度和窗口文件，使用微秒为单位。

警告：我将 hwlat 归类为微基准测试工具而不是可观察性工具，因为它执行的实验本身会扰乱系统：它会使一个 CPU 在宽度持续时间内忙碌，并禁用中断。

### 14.10 FTrace 历史触发器

Hist 触发器是 Tom Zanussi 添加到 Linux 4.7 中的一项高级 Ftrace 功能，它允许创建事件的自定义直方图。它是统计汇总的另一种形式，允许将计数按一个或多个组成部分进行细分。

单个直方图的总体用法是：

1.  `echo 'hist:_expression_' > events/.../trigger`：创建历史触发器。
    
2.  `sleep _duration_`：允许填充直方图。
    
3.  `cat events/.../hist`：打印直方图。
    
4.  `echo '!hist:_expression_' > events/.../trigger`：删除它。
    

hist 表达式的格式为：

点击这里查看代码图片

hist:keys=<field1\[,field2,...\]>\[:values=<field1\[,field2,...\]>\]
  \[:sort=<field1\[,field2,...\]>\]\[:size=#entries\]\[:pause\]\[:continue\]
  \[:clear\]\[:name=histname1\]\[:<handler>.<action>\] \[if <filter>\]

Linux 源代码中的 Documentation/trace/histogram.rst 下完整记录了该语法，以下是一些示例 \[Zanussi 20\]。

#### 14.10.1 单键

以下内容使用 hist 触发器通过 raw\_syscalls:sys\_enter 跟踪点对系统调用进行计数，并按进程 ID 提供直方图细分：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **echo 'hist:key=common\_pid' > events/raw\_syscalls/sys\_enter/trigger**
# **sleep 10**
# **cat events/raw\_syscalls/sys\_enter/hist**
# event histogram
#
# trigger info: hist:keys=common\_pid.execname:vals=hitcount:sort=hitcount:size=2048
\[active\]
#  
{ common\_pid:        347 } hitcount:          1
{ common\_pid:        345 } hitcount:          3
{ common\_pid:        504 } hitcount:          8
{ common\_pid:        494 } hitcount:         20
{ common\_pid:        502 } hitcount:         30
{ common\_pid:        344 } hitcount:         32
{ common\_pid:        348 } hitcount:         36
{ common\_pid:      32399 } hitcount:        136
{ common\_pid:      32400 } hitcount:        138
{ common\_pid:      32379 } hitcount:        177
{ common\_pid:      32296 } hitcount:        187
{ common\_pid:      32396 } hitcount:     882604  
Totals:
    Hits: 883372
    Entries: 12
    Dropped: 0
# **echo '!hist:key=common\_pid' > events/raw\_syscalls/sys\_enter/trigger**

输出显示 PID 32396 在跟踪时执行了 882,604 次系统调用，并列出了其他 PID 的计数。最后几行显示统计信息：哈希写入次数 (`Hits`)、哈希中的条目 (`Entries`) 以及条目超过哈希大小 (`Dropped`）。如果发生丢包，可以在声明时增加hash的大小；默认为 2048。

#### 14.10.2 字段

哈希字段来自事件的格式文件。对于此示例，使用了 common\_pid 字段：

点击这里查看代码图片

\# **cat events/raw\_syscalls/sys\_enter/format**
\[...\]
        field:int **common\_pid**;       offset:4;  size:4;    signed:1;

        field:long id;   offset:8;  size:8;    signed:1;
        field:unsigned long args\[6\];    offset:16;   size:48;  signed:0;

您也可以使用其他字段。对于此事件，id 字段是系统调用 ID。使用它作为哈希键：

点击这里查看代码图片

\# **echo 'hist:key=id' > events/raw\_syscalls/sys\_enter/trigger**
# cat events/raw\_syscalls/sys\_enter/hist
\[...\]
{ id:         14 } hitcount:         48
{ id:          1 } hitcount:      80362
{ id:          0 } hitcount:      80396
\[...\]

直方图显示最频繁的系统调用的 ID 为 0 和 1。在我的系统上，系统调用 ID 位于此头文件中：

点击这里查看代码图片

\# **more /usr/include/x86\_64-linux-gnu/asm/unistd\_64.h**
\[...\]
#define \_\_NR\_read 0
#define \_\_NR\_write 1
\[...\]

这表明 0 和 1 用于 read(2) 和 write(2) 系统调用。

#### 14.10.3 修饰符

由于 PID 和系统调用 ID 故障很常见，因此 hist 触发器支持注释输出的修饰符：对于 PID，使用 `.execname`；对于系统调用 ID，使用 `.syscall`。例如，将 `.execname` 修饰符添加到前面的示例中：

点击这里查看代码图片

\# **echo 'hist:key=common\_pid.execname' > events/raw\_syscalls/sys\_enter/trigger**
\[...\]
{ common\_pid: bash            \[     32379\] } hitcount:        166
{ common\_pid: sshd            \[     32296\] } hitcount:        259
{ common\_pid: dd              \[     32396\] } hitcount:     869024
\[...\]

输出现在包含进程名称，后跟方括号中的 PID，而不仅仅是 PID。

#### 14.10.4 PID 过滤器

根据前面的 by-PID 和 by-syscall ID 输出，您可以假设两者相关，并且 dd(1) 命令正在执行 read(2) 和 write(2) 系统调用。要直接测量这一点，您可以为系统调用 ID 创建直方图，然后使用过滤器来匹配 PID：

点击这里查看代码图片

\# **echo 'hist:key=id.syscall if common\_pid==32396' > \\**
    **events/raw\_syscalls/sys\_enter/trigger**
# **cat events/raw\_syscalls/sys\_enter/hist**
# event histogram
#
# trigger info: hist:keys=id.syscall:vals=hitcount:sort=hitcount:size=2048 if common\_
pid==32396 \[active\]
#

{ id: sys\_write                     \[  1\] } hitcount:     106425
{ id: sys\_read                      \[  0\] } hitcount:     106425

Totals:
    Hits: 212850
    Entries: 2
    Dropped: 0

直方图现在显示该 PID 的系统调用，并且 .syscall 修饰符已包含系统调用名称。这证实了 dd(1) 正在调用 read(2) 和 write(2)。另一个解决方案是使用多个密钥，如下一节所示。

#### 14.10.5 多个按键

以下示例包含系统调用 ID 作为第二个键：

点击这里查看代码图片

\# **echo 'hist:key=common\_pid.execname,id' > events/raw\_syscalls/sys\_enter/trigger**
# **sleep 10**
# **cat events/raw\_syscalls/sys\_enter/hist**
# event histogram
#
# trigger info: hist:keys=common\_pid.execname,id:vals=hitcount:sort=hitcount:size=2048
\[active\]
#
\[...\]
{ common\_pid: sshd            \[   14250\], id:         23 } hitcount:         36
{ common\_pid: bash            \[   14261\], id:         13 } hitcount:         42
{ common\_pid: sshd            \[   14250\], id:         14 } hitcount:         72
{ common\_pid: dd              \[   14325\], id:          0 } hitcount:    9195176
{ common\_pid: dd              \[   14325\], id:          1 } hitcount:    9195176

Totals:
    Hits: 18391064
    Entries: 75
    Dropped: 0
    Dropped: 0

输出现在显示进程名称和 PID，并按系统调用 ID 进一步细分。此输出显示 `dd` PID 142325 正在执行 ID 为 0 和 1 的两个系统调用。您可以将 `.syscall` 修饰符添加到第二个键，以使其包含系统调用名称。

#### 14.10.6 堆栈跟踪键

我经常希望知道导致该事件的代码路径，并且我建议 Tom Zanussi 添加 Ftrace 的功能，以使用整个内核堆栈跟踪作为密钥。

例如，计算导致 block:block\_rq\_issue 跟踪点的代码路径：

点击这里查看代码图片

\# **echo 'hist:key=stacktrace' > events/block/block\_rq\_issue/trigger**
# **sleep 10**
# **cat events/block/block\_rq\_issue/hist**
\[...\]
{ stacktrace:
         nvme\_queue\_rq+0x16c/0x1d0
         \_\_blk\_mq\_try\_issue\_directly+0x116/0x1c0
         blk\_mq\_request\_issue\_directly+0x4b/0xe0
         blk\_mq\_try\_issue\_list\_directly+0x46/0xb0
         blk\_mq\_sched\_insert\_requests+0xae/0x100
         blk\_mq\_flush\_plug\_list+0x1e8/0x290
         blk\_flush\_plug\_list+0xe3/0x110
         blk\_finish\_plug+0x26/0x34
         read\_pages+0x86/0x1a0
         \_\_do\_page\_cache\_readahead+0x180/0x1a0
         ondemand\_readahead+0x192/0x2d0
         page\_cache\_sync\_readahead+0x78/0xc0
         generic\_file\_buffered\_read+0x571/0xc00
         generic\_file\_read\_iter+0xdc/0x140
         ext4\_file\_read\_iter+0x4f/0x100
         new\_sync\_read+0x122/0x1b0
} hitcount:        266  
Totals:
    Hits: 522
    Entries: 10
    Dropped: 0

我已截断输出，仅显示最后、最频繁的堆栈跟踪。它表明磁盘 I/O 是通过 new\_sync\_read() 发出的，该函数调用了 ext4\_file\_read\_iter() 等等。

#### 14.10.7 综合事件

这就是事情开始变得非常奇怪的地方（如果还没有的话）。可以创建由其他事件触发的合成事件，并且可以以自定义方式组合它们的事件参数。要访问先前事件中的事件参数，可以将它们保存到直方图中并由稍后的合成事件获取。

这对于一个关键用例更有意义：自定义延迟直方图。对于合成事件，可以在一个事件上保存时间戳，然后在另一个事件上检索时间戳，以便可以计算增量时间。

例如，以下使用名为 syscall\_latency 的合成事件来计算所有系统调用的延迟，并按系统调用 ID 和名称将其显示为直方图：

点击这里查看代码图片

\# **cd /sys/kernel/debug/tracing**
# **echo 'syscall\_latency u64 lat\_us; long id' >> synthetic\_events**
# **echo 'hist:keys=common\_pid:ts0=common\_timestamp.usecs' >> \\**
    **events/raw\_syscalls/sys\_enter/trigger**
\# **echo 'hist:keys=common\_pid:lat\_us=common\_timestamp.usecs-$ts0:'\\**
    **'onmatch(raw\_syscalls.sys\_enter).trace(syscall\_latency,$lat\_us,id)' >>\\**
    **events/raw\_syscalls/sys\_exit/trigger**
# **echo 'hist:keys=lat\_us,id.syscall:sort=lat\_us' >> \\**
    **events/synthetic/syscall\_latency/trigger**
# **sleep 10**
# **cat events/synthetic/syscall\_latency/hist**
\[...\]
{ lat\_us:    5779085, id: sys\_epoll\_wait                \[232\] } hitcount:          1
{ lat\_us:    6232897, id: sys\_poll                      \[  7\] } hitcount:          1
{ lat\_us:    6233840, id: sys\_poll                      \[  7\] } hitcount:          1
{ lat\_us:    6233884, id: sys\_futex                     \[202\] } hitcount:          1
{ lat\_us:    7028672, id: sys\_epoll\_wait                \[232\] } hitcount:          1
{ lat\_us:    9999049, id: sys\_poll                      \[  7\] } hitcount:          1
{ lat\_us:   10000097, id: sys\_nanosleep                 \[ 35\] } hitcount:          1
{ lat\_us:   10001535, id: sys\_wait4                     \[ 61\] } hitcount:          1
{ lat\_us:   10002176, id: sys\_select                    \[ 23\] } hitcount:          1
\[...\]

输出被截断以仅显示最高延迟。直方图正在对延迟（以微秒为单位）和系统调用 ID 对进行计数：此输出显示 sys\_nanosleep 出现了一次 1000097 微秒的延迟。这可能显示用于设置录制持续时间的 `sleep 10` 命令。

输出也很长，因为它记录了每微秒和系统调用 ID 组合的密钥，实际上我已经超过了默认的历史记录大小 2048。您可以通过向hist 声明，或者您可以使用 `.log2` 修饰符将延迟记录为 log2。这大大减少了历史条目的数量，并且仍然具有足够的分辨率来分析延迟。

要禁用并清除此事件，请使用“！”以相反的顺序回显所有字符串。前缀。

在表 14.4 中，我用代码片段解释了这个合成事件的工作原理。

表14.4 综合事件示例说明

描述

句法

我想创建一个名为 `syscall_latency` 的合成事件，并带有两个参数：`lat_us` 和 `id`。

`echo 'syscall_latency u64 lat_us; long id' >> synthetic_events`

当sys\_enter事件发生时，以`common_pid`（当前PID）为key记录一个直方图，

`echo 'hist:keys=common_pid: ... >>   events/raw_syscalls/sys_enter/trigger`

并将当前时间（以微秒为单位）保存到与直方图键 (`common_pid`) 关联的名为 `ts0` 的直方图变量中。

`ts0=common_timestamp.usecs`

在 sys\_exit 事件中，使用 `common_pid` 作为直方图键，并且，

`echo 'hist:keys=common_pid: ... >>   events/raw_syscalls/sys_exit/trigger`

计算现在的延迟减去先前事件保存在 `ts0` 中的开始时间，并将其保存为名为 `lat_us` 的直方图变量，

`lat_us=common_timestamp.usecs-$ts0`

比较该事件和 sys\_enter 事件的直方图键。如果它们匹配（相同的 `common_pid`），则 `lat_us` 具有正确的延迟计算（对于相同的 PID，从 sys\_enter 到 sys\_exit），因此，

`onmatch(raw_syscalls.sys_enter)`

最后，使用 `lat_us` 和 `id` 作为参数触发我们的合成事件 syscall\_latency。

`.trace(syscall_latency,$lat_us,id)`

将此合成事件显示为直方图，其中 `lat_us` 和 `id` 作为字段。

`echo 'hist:keys=lat_us,id.syscall:sort=lat_us'   >> events/synthetic/syscall_latency/trigger`

Ftrace 直方图作为哈希对象（键/值存储）实现，前面的示例仅使用这些哈希进行输出：按 PID 和 ID 显示系统调用计数。对于合成事件，我们使用这些哈希值做两件额外的事情：A）存储不属于输出（时间戳）的值，B）在一个事件中，获取由另一事件设置的键/值对。我们还执行算术：减法运算。在某种程度上，我们开始编写小程序。

文档 \[Zanussi 20\] 中介绍了更多关于合成事件的内容。多年来，我一直直接或间接地向 Ftrace 和 BPF 工程师提供反馈，从我的角度来看，Ftrace 的发展是有意义的，因为它解决了我之前提出的问题。我将演变总结为：

“Ftrace 很棒，但我需要使用 BPF 按 PID 和堆栈跟踪进行计数。”

“给你，历史触发器。”

“那太好了，但我仍然需要使用 BPF 来进行自定义延迟计算。”

“给你，合成事件。”

“太好了，我写完 BPF 性能工具后会检查一下。”

“严重地？”

是的，我现在确实需要探索在某些用例中采用合成事件。它非常强大，内置于内核中，并且可以单独通过 shell 脚本使用。 （我确实读完了 BPF 书，但后来开始忙于这本书。）

### 14.11 跟踪命令

trace-cmd 是 Steven Rostedt 等人开发的开源 Ftrace 前端 \[trace-cmd 20\]。它支持用于配置跟踪系统、二进制输出格式和其他功能的子命令和选项。对于事件源，它可以使用 Ftrace 函数和 function\_graph 跟踪器，以及跟踪点和已配置的 kprobes 和 uprobes。

例如，使用trace-cmd通过函数跟踪器记录内核函数do\_nanosleep()十秒钟（使用虚拟sleep(1)命令）：

点击这里查看代码图片

\# **trace-cmd record -p function -l do\_nanosleep sleep 10**
  plugin 'function'
CPU0 data recorded at offset=0x4fe000
    0 bytes in size
CPU1 data recorded at offset=0x4fe000
    4096 bytes in size
# **trace-cmd report**
CPU 0 is empty
cpus=2
           sleep-21145 \[001\] 573259.213076: function:             do\_nanosleep
      multipathd-348   \[001\] 573259.523759: function:             do\_nanosleep
      multipathd-348   \[001\] 573260.523923: function:             do\_nanosleep
      multipathd-348   \[001\] 573261.524022: function:             do\_nanosleep
      multipathd-348   \[001\] 573262.524119: function:             do\_nanosleep
\[...\]

输出以由trace-cmd调用的sleep(1)开始（它配置跟踪，然后启动提供的命令），然后是来自multipathd PID 348的各种调用。此示例还表明trace-cmd比等效的tracefs更简洁/sys. 中的命令它也更安全：许多子命令在完成后处理清理跟踪状态。

trace-cmd 通常可以通过“trace-cmd”包安装，如果没有，可以在trace-cmd 网站 \[trace-cmd 20\] 上找到源代码。

本节展示了一系列的trace-cmd 子命令和跟踪功能。请参阅捆绑的trace-cmd 文档以了解其所有功能以及以下示例中使用的语法。

#### 14.11.1 子命令概述

Trace-cmd 的功能可通过首先指定子命令来使用，例如记录子命令的 `trace-cmd record`。表 14.5 列出了最近的 trace-cmd 版本 (2.8.3) 中的子命令选择。

表14.5 trace-cmd选择的子命令

命令

描述

`record`

跟踪并记录到trace.dat 文件

`report`

从trace.dat 文件中读取跟踪

`stream`

跟踪然后打印到标准输出

`list`

列出可用的跟踪事件

`stat`

显示内核跟踪子系统的状态

`profile`

跟踪并生成显示内核时间和延迟的自定义报告

`listen`

接受网络请求以进行跟踪

其他子命令包括 `start`、`stop`、`restart` 和 `clear`，用于控制超出单次调用 `record` 的跟踪。未来版本的trace-cmd可能会添加更多子命令；运行不带参数的 `trace-cmd` 以获得完整列表。

每个子命令都支持多种选项。这些可以用 `-h` 列出，例如，对于 record 子命令：

点击这里查看代码图片

\# **trace-cmd record -h**  
trace-cmd version 2.8.3

usage:
 trace-cmd record \[-v\]\[-e event \[-f filter\]\]\[-p plugin\]\[-F\]\[-d\]\[-D\]\[-o file\] \\
           \[-q\]\[-s usecs\]\[-O option \]\[-l func\]\[-g func\]\[-n func\] \\
           \[-P pid\]\[-N host:port\]\[-t\]\[-r prio\]\[-b size\]\[-B buf\]\[command ...\]
           \[-m max\]\[-C clock\]
          -e run command with event enabled
          -f filter for previous -e event
          -R trigger for previous -e event
          -p run command with plugin enabled
          -F filter only on the given process
          -P trace the given pid like -F for the command
          -c also trace the children of -F (or -P if kernel supports it)
          -C set the trace clock
          -T do a stacktrace on all events
          -l filter function name
          -g set graph function
          -n do not trace function
\[...\]

此输出中的选项已被截断，显示 35 个选项中的前 12 个。前 12 个包括最常用的。请注意，术语插件 (`-p`) 指的是 Ftrace 跟踪器，其中包括 function、function\_graph 和 hwlat。

#### 14.11.2 trace-cmd 单行代码

以下单行代码通过示例展示了不同的trace-cmd 功能。它们的手册页中介绍了它们的语法。

##### 上市活动

列出所有跟踪事件源和选项：

trace-cmd list

列出 Ftrace 跟踪器：

trace-cmd list -t

列出事件源（跟踪点、kprobe 事件和 uprobe 事件）：

trace-cmd list -e

列出系统调用跟踪点：

trace-cmd list -e syscalls:

显示给定跟踪点的格式文件：

点击这里查看代码图片

trace-cmd list -e syscalls:sys\_enter\_nanosleep -F

##### 函数追踪

在系统范围内跟踪内核函数：

点击这里查看代码图片

trace-cmd record -p function -l _function\_name_

跟踪系统范围内以“tcp\_”开头的所有内核函数，直到 Ctrl-C：

点击这里查看代码图片

trace-cmd record -p function -l 'tcp\_\*'

跟踪系统范围内以“tcp\_”开头的所有内核函数 10 秒：

点击这里查看代码图片

trace-cmd record -p function -l 'tcp\_\*' sleep 10

跟踪 ls(1) 命令的所有以“vfs\_”开头的内核函数：

点击这里查看代码图片

trace-cmd record -p function -l 'vfs\_\*' -F ls

跟踪 bash(1) 及其子代的所有以“vfs\_”开头的内核函数：

点击这里查看代码图片

trace-cmd record -p function -l 'vfs\_\*' -F -c bash

跟踪 PID 21124 的所有以“vfs\_”开头的内核函数

点击这里查看代码图片

trace-cmd record -p function -l 'vfs\_\*' -P 21124

##### 函数图追踪

在系统范围内跟踪内核函数及其子函数调用：

点击这里查看代码图片

trace-cmd record -p function\_graph -g _function\_name_

在系统范围内跟踪内核函数 do\_nanosleep() 和子函数 10 秒：

点击这里查看代码图片

trace-cmd record -p function\_graph -g do\_nanosleep sleep 10

##### 事件追踪

通过 sched:sched\_process\_exec 跟踪点跟踪新进程，直到 Ctrl-C：

点击这里查看代码图片

trace-cmd record -e sched:sched\_process\_exec

通过 sched:sched\_process\_exec （较短版本）跟踪新进程：

点击这里查看代码图片

trace-cmd record -e sched\_process\_exec

使用内核堆栈跟踪来跟踪块 I/O 请求：

点击这里查看代码图片

trace-cmd record -e block\_rq\_issue -T

跟踪所有块跟踪点直到 Ctrl-C：

trace-cmd record -e block

跟踪先前创建的名为“brendan”的 kprobe 10 秒：

点击这里查看代码图片

trace-cmd record -e probe:brendan sleep 10

跟踪 ls(1) 命令的所有系统调用：

点击这里查看代码图片

trace-cmd record -e syscalls -F ls

##### 报告

打印trace.dat输出文件的内容：

trace-cmd report

打印trace.dat输出文件的内容，仅限CPU 0：

trace-cmd report --cpu 0

##### 其他能力

跟踪 sched\_switch 插件的事件：

点击这里查看代码图片

trace-cmd record -p sched\_switch

监听 TCP 端口 8081 上的跟踪请求：

trace-cmd listen -p 8081

连接到远程主机以运行记录子命令：

点击这里查看代码图片

trace-cmd record ... -N _addr_:_port_

#### 14.11.3 trace-cmd 与 perf(1)

trace-cmd 子命令的风格可能会让您想起第 13 章中介绍的 perf(1)，并且这两个工具确实具有类似的功能。表 14.6 比较了 trace-cmd 和 perf(1)。

表 14.6 perf(1) 与trace-cmd

属性

性能(1)

跟踪命令

二进制输出文件

性能数据

跟踪数据

跟踪点

是的

是的

k探针

是的

部分(1)

乌鲁布斯

是的

部分(1)

泰达币

是的

部分(1)

私人管理公司

是的

不

定时采样

是的

不

函数追踪

部分(2)

是的

函数图追踪

部分(2)

是的

网络客户端/服务器

不

是的

输出文件开销

低的

很低

前端

各种各样的

内核鲨鱼

来源

在 Linux 工具/perf 中

git.kernel.org

*   部分（1）：trace-cmd 仅当这些事件已通过其他方式创建并出现在 /sys/kernel/debug/tracing/events 中时才支持这些事件。
    
*   Partial(2)：perf(1) 通过 `ftrace` 子命令支持这些，尽管它没有完全集成到 perf(1) 中（例如，它不支持 perf.data）。
    

作为相似性的示例，以下示例在系统范围内跟踪 syscalls:sys\_enter\_read 跟踪点十秒，然后使用 perf(1) 列出跟踪：

点击这里查看代码图片

\# **perf record -e syscalls:sys\_enter\_nanosleep -a sleep 10**
# **perf script**

...并使用trace-cmd：

点击这里查看代码图片

\# **trace-cmd record -e syscalls:sys\_enter\_nanosleep sleep 10**
# **trace-cmd report**

Trace-cmd 的优点之一是它更好地支持 function 和 function\_graph 跟踪器。

#### 14.11.4 跟踪cmd function\_graph

本节的开头演示了使用trace-cmd 的函数跟踪器。下面演示了同一内核函数 do\_nanosleep() 的 function\_graph 跟踪器：

点击这里查看代码图片

\# **trace-cmd record -p function\_graph -g do\_nanosleep sleep 10**
  plugin 'function\_graph'
CPU0 data recorded at offset=0x4fe000
    12288 bytes in size
CPU1 data recorded at offset=0x501000
    45056 bytes in size
# **trace-cmd report | cut -c 66-**

              |  do\_nanosleep() {
              |    hrtimer\_start\_range\_ns() {
              |      lock\_hrtimer\_base.isra.0() {
   0.250 us   |        \_raw\_spin\_lock\_irqsave();
   0.688 us   |      }
   0.190 us   |      ktime\_get();
   0.153 us   |      get\_nohz\_timer\_target();
   \[...\]

为了清楚起见，我使用 cut(1) 来隔离函数图和计时列。这截断了前面的函数跟踪示例中显示的典型跟踪字段。

#### 14.11.5 KernelShark

KernelShark 是用于 Trace-cmd 输出文件的可视化用户界面，由 Ftrace 的创建者 Steven Rostedt 创建。 KernelShark 最初是 GTK，后来由维护该项目的 Yordan Karadzhov 用 Qt 重写。 KernelShark 可以从 kernelshark 软件包（如果有）安装，或者通过其网站 \[KernelShark 20\] 上的源链接进行安装。 1.0版本是Qt版本，0.99及更早版本是GTK版本。

作为使用 KernelShark 的示例，以下记录了所有调度程序跟踪点，然后将它们可视化：

点击这里查看代码图片

\# **trace-cmd record -e 'sched:\*'**
# **kernelshark**

KernelShark 读取默认的trace-cmd 输出文件trace.dat（您可以使用`-i` 指定不同的文件）。图 14.3 显示了 KernelShark 可视化该文件。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/14fig03.jpg)

图 14.3 KernelShark

屏幕顶部显示每个 CPU 的时间线，其中任务的颜色不同。底部是事件表。 KernelShark 是交互式的：单击并向右拖动将缩放到选定的时间范围，单击并向左拖动将缩小。右键单击事件提供附加操作，例如设置过滤器。

KernelShark可用于识别由不同线程之间的交互引起的性能问题。

#### 14.11.6 Trace-cmd 文档

对于软件包安装，trace-cmd 文档应作为trace-cmd(1) 和其他手册页（例如trace-cmd-record(1)）提供，这些手册页也位于文档目录下的trace-cmd 源中。我还建议观看维护者 Steven Rostedt 关于 Ftrace 和trace-cmd 的演讲，例如“Understanding the Linux Kernel (via ftrace)”：

*   幻灯片：https://www.slideshare.net/ennael/kernel-recipes-2017-understanding-the-linux-kernel-via-ftrace-steven-rostedt
    
*   视频：https://www.youtube.com/watch?v=2ff-7UTg5rE
    

### 14.12 性能跟踪

第 13 章介绍的 perf(1) 实用程序有一个 `ftrace` 子命令，以便它可以访问函数和 function\_graph 跟踪器。

例如，在内核 do\_nanosleep() 函数上使用函数跟踪器：

点击这里查看代码图片

\# **perf ftrace -T do\_nanosleep -a sleep 10**
 0)  sleep-22821   |               |  do\_nanosleep() {
 1)  multipa-348   |               |  do\_nanosleep() {
 1)  multipa-348   | $ 1000068 us  |  }
 1)  multipa-348   |               |  do\_nanosleep() {
 1)  multipa-348   | $ 1000068 us  |  }
\[...\]

并使用 function\_graph 跟踪器：

点击这里查看代码图片

\# **perf ftrace -G do\_nanosleep -a sleep 10**
 1)  sleep-22828   |               |  do\_nanosleep() {
 1)  sleep-22828   |   ==========> |
 1)  sleep-22828   |               |    smp\_irq\_work\_interrupt() {
 1)  sleep-22828   |               |      irq\_enter() {
 1)  sleep-22828   |   0.258 us    |        rcu\_irq\_enter();
 1)  sleep-22828   |   0.800 us    |      }
 1)  sleep-22828   |               |      \_\_wake\_up() {
 1)  sleep-22828   |               |        \_\_wake\_up\_common\_lock() {
 1)  sleep-22828   |   0.491 us    |          \_raw\_spin\_lock\_irqsave();
\[...\]

ftrace 子命令支持一些选项，包括用于匹配 PID 的 `-p`。这是一个简单的包装器，不与其他 perf(1) 功能集成：例如，它将跟踪输出打印到 stdout 并且不使用 perf.data 文件。

### 14.13 性能工具

perf-tools 是我自己开发的基于 Ftrace 和 perf(1) 的高级性能分析工具的开源集合，默认安装在 Netflix \[Gregg 20i\] 的服务器上。我设计这些工具是为了易于安装（几乎没有依赖项）并且易于使用：每个工具都应该做一件事并且做得很好。 perf-tools 本身主要以 shell 脚本的形式实现，可以自动设置 tracefs /sys 文件。

例如，使用 execsnoop(8) 来跟踪新进程：

点击这里查看代码图片

\# **execsnoop**
Tracing exec()s. Ctrl-C to end.
   PID   PPID ARGS
  6684   6682 cat -v trace\_pipe
  6683   6679 gawk -v o=1 -v opt\_name=0 -v name= -v opt\_duration=0 \[...\]
  6685  20997 man ls
  6695   6685 pager
  6691   6685 preconv -e UTF-8
  6692   6685 tbl
  6693   6685 nroff -mandoc -rLL=148n -rLT=148n -Tutf8
  6698   6693 locale charmap
  6699   6693 groff -mtty-char -Tutf8 -mandoc -rLL=148n -rLT=148n
  6700   6699 troff -mtty-char -mandoc -rLL=148n -rLT=148n -Tutf8
  6701   6699 grotty
\[...\]

此输出首先显示 excesnoop(8) 本身使用的 cat(1) 和 gawk(1) 命令，然后是 `man ls` 执行的命令。它可用于调试其他工具不可见的短期进程问题。

execsnoop(8) 支持的选项包括用于时间戳的 `-t` 和用于总结命令行用法的 `-h`。 execsnoop(8) 和所有其他工具也有一个手册页和一个示例文件。

#### 14.13.1 工具覆盖范围

图 14.4 显示了不同的性能工具以及它们可以观察的系统区域。

![Images](/Systems%20Performance_%20Enterprise%20and%20the%20Cl%20-%20Brendan%20Gregg/images/14fig04.jpg)

图 14.4 性能工具

许多是单一用途的工具，带有单个箭头；有些是左侧列出的多用途工具，并带有双箭头以显示其覆盖范围。

#### 14.13.2 单一用途工具

单一用途工具在图 14.4 中用单箭头显示。其中一些已在前面的章节中介绍过。

诸如 execsnoop(8) 之类的单一用途工具只做一项工作并且做得很好（Unix 哲学）。此设计包括使默认输出简洁且通常足够，这有助于帮助学习。您可以“仅运行 execsnoop”，无需学习任何命令行选项，并获得足够的输出来解决您的问题，而不会造成不必要的混乱。通常确实存在用于定制的选项。

表 14.7 中描述了单一用途工具。

表 14.7 单一用途性能工具

工具

用途

描述

一口大小(8)

性能

将磁盘 I/O 大小汇总为直方图

缓存统计(8)

追踪

显示页面缓存命中/未命中统计信息

执行监听(8)

追踪

使用参数跟踪新进程（通过 execve(2)）

延迟(8)

追踪

将磁盘 I/O 延迟总结为直方图

iosnoop(8)

追踪

跟踪磁盘 I/O 并提供包括延迟在内的详细信息

杀戮窥探(8)

追踪

跟踪kill(2)信号显示进程和信号详细信息

打开监听(8)

追踪

跟踪显示文件名的 open(2) 系列系统调用

tcpretrans(8)

追踪

跟踪 TCP 重传，显示地址和内核状态

execsnoop(8) 之前已演示过。作为另一个示例，iolatency(8) 将磁盘 I/O 延迟显示为直方图：

点击这里查看代码图片

\# **iolatency**
Tracing block I/O. Output every 1 seconds. Ctrl-C to end.

  >=(ms) .. <(ms)   : I/O      |Distribution                          |
       0 -> 1       : 731      |######################################|
       1 -> 2       : 318      |#################                     |
       2 -> 4       : 160      |#########                             |

  >=(ms) .. <(ms)   : I/O      |Distribution                          |
       0 -> 1       : 2973     |######################################|
       1 -> 2       : 497      |#######                               |
       2 -> 4       : 26       |#                                     |
       4 -> 8       : 3        |#                                     |

  >=(ms) .. <(ms)   : I/O      |Distribution                          |
       0 -> 1       : 3130     |######################################|
       1 -> 2       : 177      |###                                   |
       2 -> 4       : 1        |#                                     |
^C

此输出显示 I/O 延迟通常较低，在 0 到 1 毫秒之间。

我实现这一点的方式有助于解释扩展 BPF 的需求。 iolatency(8) 跟踪块 I/O 问题和完成跟踪点，读取用户空间中的所有事件，解析它们，并使用 awk(1) 将它们后处理为这些直方图。由于大多数服务器上磁盘 I/O 的频率相对较低，因此这种方法无需繁重的开销即可实现。但对于更频繁的事件（例如网络 I/O 或调度）来说，开销将令人望而却步。扩展BPF解决了这个问题，它允许在内核空间计算直方图摘要，并且只将摘要传递到用户空间，大大减少了开销。 Ftrace 现在支持一些带有 hist 触发器和合成事件的类似功能，如第 14.10 节 Ftrace Hist 触发器中所述（我需要更新 iolatency(8) 才能使用它们）。

我确实为自定义直方图开发了一个预 BPF 解决方案，并将其公开为 perf-stat-hist(8) 多功能工具。

#### 14.13.3 多用途工具

图 14.4 列出并描述了多用途工具。它们支持多个事件源并且可以执行许多角色，类似于 perf(1) 和trace-cmd，尽管这也使它们使用起来很复杂。

表 14.8 多用途性能工具

工具

用途

描述

函数计数(8)

追踪

计算内核函数调用次数

函数图(8)

追踪

跟踪内核函数显示子函数代码流程

函数跟踪(8)

追踪

跟踪核函数

功能较慢(8)

追踪

跟踪慢于阈值的内核函数

探针(8)

追踪

内核函数的动态跟踪

性能统计历史记录(8)

性能(1)

跟踪点参数的自定义幂聚合

系统计数(8)

性能(1)

总结系统调用

点(8)

追踪

跟踪跟踪点

探头(8)

追踪

用户级函数的动态跟踪

为了帮助使用这些工具，您可以收集并分享俏皮话。我在下一节中提供了它们，类似于我的 perf(1) 和trace-cmd 的单行部分。

#### 14.13.4 perf-tools 单行工具

除非另有说明，否则以下单行语句将在系统范围内进行跟踪，直到键入 Ctrl-C。它们分为使用 Ftrace 分析、Ftrace 跟踪器和事件跟踪（跟踪点、kprobes、uprobes）的那些。

##### Ftrace 分析器

统计所有内核 TCP 函数：

funccount 'tcp\_\*'

统计所有内核 VFS 函数，每 1 秒打印前 10 个函数：

funccount -t 10 -i 1 'vfs\*'

##### Ftrace 追踪器

跟踪内核函数 do\_nanosleep() 并显示所有子调用：

funcgraph do\_nanosleep

跟踪内核函数 do\_nanosleep() 并显示最多 3 层深度的子调用：

funcgraph -m 3 do\_nanosleep

计算 PID 198 的所有以“sleep”结尾的内核函数：

functrace -p 198 '\*sleep'

跟踪 vfs\_read() 调用速度慢于 10 毫秒：

funcslower vfs\_read 10000

##### 事件追踪

使用 kprobe 跟踪 do\_sys\_open() 内核函数：

kprobe p:do\_sys\_open

使用 kretprobe 跟踪 do\_sys\_open() 的返回，并打印返回值：

点击这里查看代码图片

kprobe 'r:do\_sys\_open $retval'

跟踪 do\_sys\_open() 的文件模式参数：

点击这里查看代码图片

kprobe 'p:do\_sys\_open mode=$arg3:u16'

跟踪 do\_sys\_open() 的文件模式参数（特定于 x86\_64）：

点击这里查看代码图片

kprobe 'p:do\_sys\_open mode=%dx:u16'

将 do\_sys\_open() 的文件名参数跟踪为字符串：

点击这里查看代码图片

kprobe 'p:do\_sys\_open filename=+0($arg2):string'

将 do\_sys\_open() （特定于 x86\_64）的文件名参数跟踪为字符串：

点击这里查看代码图片

kprobe 'p:do\_sys\_open filename=+0(%si):string'

当文件名匹配“\*stat”时跟踪 do\_sys\_open()：

点击这里查看代码图片

kprobe 'p:do\_sys\_open file=+0($arg2):string' 'file ~ "\*stat"'

使用内核堆栈跟踪来跟踪 tcp\_retransmit\_skb()：

点击这里查看代码图片

kprobe -s p:tcp\_retransmit\_skb

列出跟踪点：

tpoint -l

使用内核堆栈跟踪跟踪磁盘 I/O：

点击这里查看代码图片

tpoint -s block:block\_rq\_issue

跟踪所有“bash”可执行文件中的用户级 readline() 调用：

uprobe p:bash:readline

跟踪 readline() 从“bash”的返回并将其返回值打印为字符串：

点击这里查看代码图片

uprobe 'r:bash:readline +0($retval):string'

跟踪 /bin/bash 中的 readline() 条目及其条目参数 (x86\_64) 作为字符串：

点击这里查看代码图片

uprobe 'p:/bin/bash:readline prompt=+0(%di):string'

仅跟踪 PID 1234 的 libc gettimeofday() 调用：

点击这里查看代码图片

uprobe -p 1234 p:libc:gettimeofday

仅当 fopen() 返回 NULL（并使用“文件”别名）时才跟踪 fopen() 的返回：

点击这里查看代码图片

uprobe 'r:libc:fopen file=$retval' 'file == 0'

##### CPU寄存器

函数参数别名 ($arg1, ..., $argN) 是较新的 Ftrace 功能 (Linux 4.20+)。对于较旧的内核（或缺少别名的处理器体系结构），您将需要使用 CPU 寄存器名称，如第 14.6.2 节“参数”中所述。这些单行代码包括一些 x86\_64 寄存器（%di、%si、%dx）作为示例。调用约定记录在 syscall(2) 手册页中：

点击这里查看代码图片

$ **man 2 syscall**
\[...\]
       Arch/ABI      arg1  arg2  arg3  arg4  arg5  arg6  arg7  Notes
       ──────────────────────────────────────────────────────────────
\[...\]
       sparc/32      o0    o1    o2    o3    o4    o5    -
       sparc/64      o0    o1    o2    o3    o4    o5    -
       tile          R00   R01   R02   R03   R04   R05   -
       **x86-64        rdi   rsi   rdx**   r10   r8    r9    -
       x32           rdi   rsi   rdx   r10   r8    r9    -
\[...\]

#### 14.13.5 示例

作为使用工具的示例，以下使用 funccount(8) 来统计 VFS 调用（与“vfs\_\*”匹配的函数名称）：

点击这里查看代码图片

\# **funccount 'vfs\_\*'**
Tracing "vfs\_\*"... Ctrl-C to end.
^C
FUNC                              COUNT
vfs\_fsync\_range                      10
vfs\_statfs                           10
vfs\_readlink                         35
vfs\_statx                           673
vfs\_write                           782
vfs\_statx\_fd                        922
vfs\_open                           1003
vfs\_getattr                        1390
vfs\_getattr\_nosec                  1390
vfs\_read                           2604

此输出显示，在跟踪期间，vfs\_read() 被调用了 2,604 次。我经常使用 funccount(8) 来确定哪些内核函数被频繁调用，哪些被调用。由于它的开销相对较低，我可以用它来检查函数调用率是否足够低以进行更昂贵的跟踪。

#### 14.13.6 性能工具与 BCC/BPF

我最初为 Netflix 云开发 perf-tools，当时它运行 Linux 3.2，缺乏扩展的 BPF。从那时起，Netflix 已转向更新的内核，我重写了其中许多工具以使用 BPF。例如，perf-tools 和 BCC 都有自己的 funccount(8)、execsnoop(8)、opensnoop(8) 等版本。

BPF 提供了可编程性和更强大的功能，第 15 章介绍了 BCC 和 bpftrace BPF 前端。不过，perf-tools[^5] 有一些优点：

*   funccount(8)：perf-tools 版本使用 Ftrace 函数分析，与 BCC 中当前基于 kprobe 的 BPF 版本相比，效率更高且约束更少。
    
*   funcgraph(8)：BCC 中不存在该工具，因为它使用 Ftrace function\_graph 跟踪。
    
*   Hist Triggers：这将为未来的性能工具提供动力，这些工具应该比基于 kprobe 的 BPF 版本更高效。
    
*   依赖性：性能工具对于资源受限的环境（例如嵌入式 Linux）仍然有用，因为它们通常只需要 shell 和 awk(1)。
    

[^5]我原本以为当我们完成 BPF 跟踪后我就会停用 perf-tools，但由于这些原因我还是让它继续存在。

我有时还使用 perf-tools 工具来交叉检查和调试 BPF 工具的问题。[^6]

[^6]我可以重新利用一句名言：拥有一根追踪器的人知道发生了什么；一个拥有两个示踪剂的人知道其中一个已损坏，并搜索 lkml 希望找到补丁。

#### 14.13.7 文档

工具通常有一个使用消息来总结其语法。例如：

点击这里查看代码图片

\# **funccount -h**
USAGE: funccount \[-hT\] \[-i secs\] \[-d secs\] \[-t top\] funcstring
                 -d seconds      # total duration of trace
                 -h              # this usage message
                 -i seconds      # interval summary
                 -t top          # show top num entries only
                 -T              # include timestamp (for -i)
  eg,
       funccount 'vfs\*'          # trace all funcs that match "vfs\*"
       funccount -d 5 'tcp\*'     # trace "tcp\*" funcs for 5 seconds
       funccount -t 10 'ext3\*'   # show top 10 "ext3\*" funcs
       funccount -i 1 'ext3\*'    # summary every 1 second
       funccount -i 1 -d 5 'ext3\*' # 5 x 1 second summaries

每个工具在 perf-tools 存储库 (funccount\_example.txt) 中还有一个手册页和一个示例文件，其中包含带有注释的输出示例。

### 14.14 FTrace 文档

Ftrace（和跟踪事件）在 Linux 源代码的 Documentation/trace 目录下有详细记录。该文档也在线：

*   https://www.kernel.org/doc/html/latest/trace/ftrace.html
    
*   https://www.kernel.org/doc/html/latest/trace/kprobetrace.html
    
*   https://www.kernel.org/doc/html/latest/trace/uprobetracer.html
    
*   https://www.kernel.org/doc/html/latest/trace/events.html
    
*   https://www.kernel.org/doc/html/latest/trace/histogram.html
    

前端资源有：

*   跟踪cmd：https://trace-cmd.org
    
*   perf ftrace：在 Linux 源中：tools/perf/Documentation/perf-ftrace.txt
    
*   性能工具：https://github.com/brendangregg/perf-tools
    

### 14.15 参考文献

\[Rostedt 08\] Rostedt, S.，“ftrace - 函数跟踪器”，Linux 文档，https://www.kernel.org/doc/html/latest/trace/ftrace.html，2008 年以上。

\[Matz 13\] Matz, M.、Hubička, J.、Jaeger, A. 和 Mitchell, M.，“System V 应用程序二进制接口，AMD64 架构处理器补充，草案版本 0.99.6”，http://x86- 64.org/documentation/abi.pdf，2013 年。

\[Gregg 19f\] Gregg, B.，“两个内核之谜和我见过的最具技术性的演讲”，http://www.brendangregg.com/blog/2019-10-15/kernelrecipes-kernel-ftrace-internals .html，2019。

\[Dronamraju 20\] Dronamraju, S.，“Uprobe-tracer：基于 Uprobe 的事件跟踪”，Linux 文档，https://www.kernel.org/doc/html/latest/trace/uprobetracer.html，2020 年访问。

\[Gregg 20i\] Gregg, B.，“基于 Linux perf\_events（又名 perf）和 ftrace 的性能分析工具”，https://github.com/brendangregg/perf-tools，最后更新于 2020 年。

\[Hiramatsu 20\] Hiramatsu, M.，“基于 Kprobe 的事件跟踪”，Linux 文档，https://www.kernel.org/doc/html/latest/trace/kprobetrace.html，2020 年访问。

\[KernelShark 20\]“KernelShark”，https://www.kernelshark.org，2020 年访问。

\[trace-cmd 20\]“TRACE-CMD”，https://trace-cmd.org，2020 年访问。

\[Ts'o 20\] Ts'o, T.、Zefan, L. 和 Zanussi, T.，“事件跟踪”，Linux 文档，https://www.kernel.org/doc/html/latest/trace/ events.html，2020 年访问。

\[Zanussi 20\] Zanussi, T.，“事件直方图”，Linux 文档，https://www.kernel.org/doc/html/latest/trace/histogram.html，2020 年访问。