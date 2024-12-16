# 附录E

系统性能名人录

了解谁创造了我们使用的技术可能很有用。这是基于本书中的技术的系统性能领域的名人列表。这是受到 \[Libes 89\] 中列出的 Unix 名人录的启发。向那些失踪或被盗用的人致歉。如果您想进一步深入了解人员和历史，请参阅 Linux 源代码中列出的章节参考和名称、Linux 存储库历史记录和 Linux 源代码中的 MAINTAINERS 文件。我的 BPF 书 \[Gregg 19\] 的致谢部分还列出了各种技术，特别是扩展的 BPF、BCC、bpftrace、kprobes 和 uprobes，以及它们背后的人员。

John Allspaw：容量规划 \[Allspaw 08\]。

Gene M. Amdahl：计算机可扩展性的早期工作 \[Amdahl 67\]。

Jens Axboe：CFQ I/O 调度程序、fio、blktrace、io\_uring。

布伦登·布兰科：密件抄送。

Jeff Bonwick：发明了内核slab分配，共同发明了用户级slab分配，共同发明了ZFS、kstat，首先开发了mpstat。

Daniel Borkmann：扩展 BPF 的共同创建者和维护者。

Roch Bourbonnais：Sun Microsystems 系统性能专家。

Tim Bray：编写了 Bonnie 磁盘 I/O 微基准测试，以 XML 闻名。

Bryan Cantrill：DTrace 共同创建者； Oracle ZFS 存储设备分析。

Rémy Card：ext2 和 ext3 文件系统的主要开发者。

Nadia Yvette Chambers：Linux巨大的文件系统。

Guillaume Chazarain：适用于 Linux 的 iotop(1)。

Adrian Cockcroft：性能书籍 \[Cockcroft 95\]\[Cockcroft 98\]，虚拟 Adrian（SE 工具包）。

Tim Cook：适用于 Linux 的 nicstat(1) 以及增强功能。

Alan Cox：Linux 网络堆栈性能。

Mathieu Desnoyers：Linux Trace Toolkit (LTTng)，内核跟踪点，用户空间 RCU 的主要作者。

弗兰克·查. Eigler：SystemTap 的首席开发人员。

Richard Elling：静态性能调优方法。

Julia Evans：性能和调试文档和工具。

凯文·罗伯特·埃尔兹：DNLC。

Roger Faulkner：为 UNIX System V 编写了 /proc，为 Solaris 编写了线程实现，以及 truss(1) 系统调用跟踪器。

Thomas Gleixner：各种 Linux 内核性能工作，包括 hrtimers。

Sebastian Godard：Linux 的 sysstat 软件包，其中包含许多性能工具，包括 iostat(1)、mpstat(1)、pidstat(1)、nfsiostat(1)、cifsiostat(1) 以及 sar(1)、sadc 的增强版(8)、sadf(1)（参见附录 B 中的指标）。

Sasha Goldshtein：BPF 工具（argdist(8)、trace(8) 等）、BCC 贡献。

Brendan Gregg：nicstat(1)、DTraceToolkit、ZFS L2ARC、BPF 工具（execsnoop、biosnoop、ext4slower、tcptop 等）、BCC/bpftrace 贡献、USE 方法、热图（延迟、利用率、亚秒偏移）、火焰图，火焰范围，本书和以前的书 \[Gregg 11a\]\[Gregg 19\]，其他性能工作。

Neil Gunther 博士：通用可扩展性定律、CPU 利用率的三元图、性能书籍 \[Gunther 97\]。

Jeffrey Hollingsworth：动态仪器 \[Hollingsworth 94\]。

Van Jacobson：traceroute(8)、pathchar、TCP/IP 性能。

Raj Jain：系统性能理论 \[Jain 91\]。

杰里·耶利内克：Solaris 区域。

Bill Joy：vmstat(1)、BSD 虚拟内存工作、TCP/IP 性能、FFS。

Andi Kleen：英特尔的性能，对 Linux 的众多贡献。

Christoph Lameter：SLUB 分配器。

William LeFebvre：编写了 top(1) 的第一个版本，启发了许多其他工具。

David Levinthal：英特尔处理器性能专家。

约翰·莱文：OProfile。

Mike Loukides：第一本关于 Unix 系统性能的书 \[Loukides 90\]，它开始或鼓励了基于资源的分析传统：CPU、内存、磁盘、网络。

Robert Love：Linux 内核性能工作，包括抢占。

Mary Marchini：libstapsdt：各种语言的动态 USDT。

Jim Mauro：Solaris 性能和工具 \[McDougall 06a\]、DTrace：Oracle Solaris、Mac OS X 和 FreeBSD 中的动态跟踪 \[Gregg 11\] 的合著者。

Richard McDougall：Solaris 微观状态会计，《Solaris 性能和工具》\[McDougall 06a\] 的合著者。

Marshall Kirk McKusick：FFS，致力于 BSD。

Arnaldo Carvalho de Melo：Linux perf(1) 维护者。

Barton Miller：动态仪表 \[Hollingsworth 94\]。

David S. Miller：Linux 网络维护者和 SPARC 维护者。许多性能改进，并支持扩展 BPF。

卡里·米尔萨普：方法 R。

Ingo Molnar：O(1) 调度程序、完全公平调度程序、自愿内核抢占、ftrace、perf，以及实时抢占、互斥体、futexes、调度程序分析、工作队列等方面的工作。

理查德·J·摩尔：DProbes、kprobes。

安德鲁·莫顿：fadvise，预读。

Gian-Paolo D. Musumeci：系统性能调优，第二版。 \[穆苏梅吉 02\]。

迈克·穆斯：ping(8)。

Shailabh Nagar：延迟会计、任务统计。

Rich Pettit：SE 工具包。

Nick Piggin：Linux 调度程序领域。

Bill Pijewski：Solaris vfsstat(1M)、ZFS I/O 限制。

Dennis Ritchie：Unix，及其原始性能特征：进程优先级、交换、缓冲区高速缓存等。

阿拉斯泰尔·罗伯逊：创建了 bpftrace。

Steven Rostedt：Ftrace、KernelShark、实时 Linux、自适应旋转互斥体、Linux 跟踪支持。

Rusty Russell：原始 futexes，各种 Linux 内核工作。

Michael Shapiro：DTrace 的共同创建者。

Aleksey Shipilëv：Java 性能专家。

Balbir Singh：Linux 内存资源控制器、延迟统计、taskstats、cgroupstats、CPU 统计。

宋永红：BTF，并扩展了BPF和BCC工作。

Alexei Starovoitov：扩展 BPF 的共同创建者和维护者。

Ken Thompson：Unix 及其原始性能特征：进程优先级、交换、缓冲区高速缓存等。

马丁·汤普森：机械同情。

Linus Torvalds：Linux 内核和系统性能所需的众多核心组件、Linux I/O 调度程序、Git。

Arjan van de Ven：latencytop、PowerTOP、irqbalance，从事 Linux 调度程序分析工作。

Nitsan Wakart：Java 性能专家。

Tobias Waldekranz：ply（第一个高级 BPF 跟踪器）。

你好，维尔斯：dstat。

Karim Yaghmour：LTT，推动 Linux 中的跟踪。

Jovi Zhangwei: ktap.

Tom Zanussi：追踪历史触发器。

Peter Zijlstra：自适应旋转互斥体实现、hardirq 回调框架、其他 Linux 性能工作。

## E.1 参考文献

\[Amdahl 67\] Amdahl, G.，“实现大规模计算能力的单处理器方法的有效性”，AFIPS，1967 年。

\[Libes 89\] Libes, D. 和 Ressler, S.，《UNIX 生活：每个人的指南》，Prentice Hall，1989 年。

\[Loukides 90\] Loukides, M.，系统性能调优，O’Reilly，1990。

\[Hollingsworth 94\] Hollingsworth, J.、Miller, B. 和 Cargille, J.，“用于可扩展性能工具的动态程序检测”，可扩展高性能计算会议 (SHPCC)，1994 年 5 月。

\[Cockcroft 95\] Cockcroft, A.，Sun 性能和调优，Prentice Hall，1995。

\[Cockcroft 98\] Cockcroft, A. 和 Pettit, R.，Sun 性能和调优：Java 和 Internet，Prentice Hall，1998 年。

\[Musumeci 02\] Musumeci, G. D. 和 Loukidas, M.，系统性能调优，第 2 版，O’Reilly，2002 年。

\[McDougall 06a\] McDougall, R.、Mauro, J. 和 Gregg, B.，Solaris 性能和工具：Solaris 10 和 OpenSolaris 的 DTrace 和 MDB 技术，Prentice Hall，2006 年。

\[Gunther 07\] Gunther, N.，游击队能力规划，Springer，2007 年。

\[Allspaw 08\] Allspaw, J.，《容量规划的艺术》，O’Reilly，2008 年。

\[Gregg 11a\] Gregg, B. 和 Mauro, J.，DTrace：Oracle Solaris、Mac OS X 和 FreeBSD 中的动态跟踪，Prentice Hall，2011 年。

\[Gregg 19\] Gregg, B.，BPF 性能工具：Linux 系统和应用程序可观察性，Addison-Wesley，2019 年。