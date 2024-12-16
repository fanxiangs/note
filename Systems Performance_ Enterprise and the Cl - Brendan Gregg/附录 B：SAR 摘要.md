    

# 附录B

SAR总结

这是系统活动报告器 sar(1) 的选项和指标的摘要。您可以使用它来唤起您的记忆，哪些指标属于哪些选项。请参阅手册页以获取完整列表。

sar(1) 在第 4 章“可观测性工具”第 4.4 节中介绍，所选选项在后面的章节（6、7、8、9、10）中进行了总结。

选项

指标

描述

`-u-P ALL`

`**%user** %nice **%system %iowait %steal %idle**`

每个 CPU 利用率（`-u` 可选）

`-u`

`**%user** %nice **%system %iowait %steal %idle**`

CPU利用率

`-u ALL`

`**... %irq %soft** %guest %gnice`

CPU 利用率扩展

`-m CPU-P ALL`

`**MHz**`

每个 CPU 的频率

`-q`

`**runq-sz** plist-sz ldavg-1 ldavg-5 ldavg-15 **blocked**`

CPU运行队列大小

`-w`

`**proc/s cswch/s**`

CPU 调度程序事件

`-B`

`pgpgin/s pgpgout/s fault/s majflt/s pgfree/s **pgscank/s pgscand/s** pgsteal/s %vmeff`

分页统计

`-H`

`kbhugfree kbhugused %hugused`

大页面

`-r`

`kbmemfree **kbavail** kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty`

内存利用率

`-S`

`kbswpfree kbswpused **%swpused** kbswpcad %swpcad`

掉期利用率

`-W`

`**pswpin/s pswpout/s**`

交换统计数据

`-v`

`dentunusd file-nr inode-nr pty-nr`

内核表

`-d`

`**tps rkB/s wkB/s** areq-sz aqu-sz **await** svctm **%util**`

磁盘统计

`-n DEV`

`**rxpck/s txpck/s rxkB/s txkB/s** rxcmp/s txcmp/s rxmcst/s **%ifutil**`

网络接口统计

`-n EDEV`

`**rxerr/s txerr/s coll/s rxdrop/s txdrop/s** txcarr/s rxfram/s rxfifo/s txfifo/s`

网络接口错误

`-n IP`

`irec/s fwddgm/s idel/s orq/s asmrq/s asmok/s fragok/s fragcrt/s`

IP统计

`-n EIP`

`ihdrerr/s iadrerr/s iukwnpr/s idisc/s odisc/s onort/s asmf/s fragf/s`

IP错误

`-n TCP`

`**active/s passive/s iseg/s oseg/s**`

TCP统计

`-n ETCP`

`atmptf/s estres/s **retrans/s** isegerr/s orsts/s`

TCP 错误

`-n SOCK`

`totsck tcpsck udpsck rawsck ip-frag tcp-tw`

套接字统计

我以粗体突出显示了我寻找的关键指标。

某些 sar(1) 选项可能需要启用内核功能（例如大页），并且在 sar(1) 的更高版本中添加了一些指标（此处显示了版本 12.0.6）。