# 附录C

bpftrace 单行线

本附录包含一些方便的 bpftrace 行话。除了本身有用之外，它们还可以帮助您一次一行地学习 bpftrace。其中大部分已包含在前面的章节中。许多可能不会立即工作：它们可能取决于某些跟踪点或函数的存在，或者取决于特定的内核版本或配置。

有关 bpftrace 的介绍，请参阅第 15 章第 15.2 节。

## 中央处理器

使用参数跟踪新进程：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_execve { join(args->argv); }'

按进程计数系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:raw\_syscalls:sys\_enter { @\[pid, comm\] = count(); }'

按系统调用探针名称计数系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_\* { @\[probe\] = count(); }'

99 赫兹运行进程名称示例：

点击这里查看代码图片

bpftrace -e 'profile:hz:99 { @\[comm\] = count(); }'

系统范围内 49 赫兹的示例用户和内核堆栈，进程名称为：

点击这里查看代码图片

bpftrace -e 'profile:hz:49 { @\[kstack, ustack, comm\] = count(); }'

PID 189、49 赫兹的用户级堆栈示例：

点击这里查看代码图片

bpftrace -e 'profile:hz:49 /pid == 189/ { @\[ustack\] = count(); }'

对于 PID 189，49 赫兹下 5 帧深的示例用户级堆栈：

点击这里查看代码图片

bpftrace -e 'profile:hz:49 /pid == 189/ { @\[ustack(5)\] = count(); }'

对于名为“mysqld”的进程，49 赫兹的用户级堆栈示例：

点击这里查看代码图片

bpftrace -e 'profile:hz:49 /comm == "mysqld"/ { @\[ustack\] = count(); }'

计算内核 CPU 调度程序跟踪点：

点击这里查看代码图片

bpftrace -e 'tracepont:sched:\* { @\[probe\] = count(); }'

计算上下文切换事件的 CPU 外内核堆栈数：

点击这里查看代码图片

bpftrace -e 'tracepont:sched:sched\_switch { @\[kstack\] = count(); }'

计算以“vfs\_”开头的内核函数调用：

点击这里查看代码图片

bpftrace -e 'kprobe:vfs\_\* { @\[func\] = count(); }'

通过 pthread\_create() 跟踪新线程：

点击这里查看代码图片

bpftrace -e 'u:/lib/x86\_64-linux-gnu/libpthread-2.27.so:pthread\_create {
    printf("%s by %s (%d)\\n", probe, comm, pid); }'

## 记忆

按用户堆栈和进程对 libc malloc() 请求字节求和（高开销）：

点击这里查看代码图片

bpftrace -e 'u:/lib/x86\_64-linux-gnu/libc.so.6:malloc {
    @\[ustack, comm\] = sum(arg0); }'

按用户堆栈对 PID 181 的 libc malloc() 请求字节求和（高开销）：

点击这里查看代码图片

bpftrace -e 'u:/lib/x86\_64-linux-gnu/libc.so.6:malloc /pid == 181/ {
    @\[ustack\] = sum(arg0); }'

将 PID 181 的用户堆栈的 libc malloc() 请求字节显示为 2 的幂直方图（高开销）：

点击这里查看代码图片

bpftrace -e 'u:/lib/x86\_64-linux-gnu/libc.so.6:malloc /pid == 181/ {
    @\[ustack\] = hist(arg0); }'

通过内核堆栈跟踪求和内核 kmem 缓存分配字节数：

点击这里查看代码图片

bpftrace -e 't:kmem:kmem\_cache\_alloc { @bytes\[kstack\] = sum(args->bytes\_alloc); }'

按代码路径计算进程堆扩展 (brk(2))：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_brk { @\[ustack, comm\] = count(); }'

按进程计数页面错误：

点击这里查看代码图片

bpftrace -e 'software:page-fault:1 { @\[comm, pid\] = count(); }'

通过用户级堆栈跟踪来计算用户页面错误：

点击这里查看代码图片

bpftrace -e 't:exceptions:page\_fault\_user { @\[ustack, comm\] = count(); }'

按跟踪点对 vmscan 操作进行计数：

点击这里查看代码图片

bpftrace -e 'tracepoint:vmscan:\* { @\[probe\]++; }'

按进程计数交换：

点击这里查看代码图片

bpftrace -e 'kprobe:swap\_readpage { @\[comm, pid\] = count(); }'

计算页面迁移：

点击这里查看代码图片

bpftrace -e 'tracepoint:migrate:mm\_migrate\_pages { @ = count(); }'

跟踪压缩事件：

点击这里查看代码图片

bpftrace -e 't:compaction:mm\_compaction\_begin { time(); }'

列出 libc 中的 USDT 探针：

点击这里查看代码图片

bpftrace -l 'usdt:/lib/x86\_64-linux-gnu/libc.so.6:\*'

列出内核 kmem 跟踪点：

bpftrace -l 't:kmem:\*'

列出所有内存子系统 (mm) 跟踪点：

bpftrace -l 't:\*:mm\_\*'

## 文件系统

通过 openat(2) 打开的跟踪文件，进程名称为：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_openat { printf("%s %s\\n", comm,
    str(args->filename)); }'

按系统调用类型计数读取系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_\*read\* { @\[probe\] = count(); }'

按系统调用类型计算写入系统调用：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_\*write\* { @\[probe\] = count(); }'

显示 read() 系统调用请求大小的分布：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_enter\_read { @ = hist(args->count); }'

显示 read() 系统调用读取字节（和错误）的分布：

点击这里查看代码图片

bpftrace -e 'tracepoint:syscalls:sys\_exit\_read { @ = hist(args->ret); }'

按错误代码计数 read() 系统调用错误：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_exit\_read /args->ret < 0/ { @\[- args->ret\] = count(); }'

计算 VFS 调用数：

点击这里查看代码图片

bpftrace -e 'kprobe:vfs\_\* { @\[probe\] = count(); }'

统计 PID 181 的 VFS 调用：

点击这里查看代码图片

bpftrace -e 'kprobe:vfs\_\* /pid == 181/ { @\[probe\] = count(); }'

计算 ext4 跟踪点：

点击这里查看代码图片

bpftrace -e 'tracepoint:ext4:\* { @\[probe\] = count(); }'

计算 xfs 跟踪点：

点击这里查看代码图片

bpftrace -e 'tracepoint:xfs:\* { @\[probe\] = count(); }'

按进程名称和用户级堆栈统计 ext4 文件读取次数：

点击这里查看代码图片

bpftrace -e 'kprobe:ext4\_file\_read\_iter { @\[ustack, comm\] = count(); }'

跟踪 ZFS spa\_sync() 次数：

点击这里查看代码图片

bpftrace -e 'kprobe:spa\_sync { time("%H:%M:%S ZFS spa\_sync()\\n"); }'

按进程名称和 PID 计算 dcache 引用：

点击这里查看代码图片

bpftrace -e 'kprobe:lookup\_fast { @\[comm, pid\] = count(); }'

## 磁盘

计算块 I/O 跟踪点事件：

点击这里查看代码图片

bpftrace -e 'tracepoint:block:\* { @\[probe\] = count(); }'

将块 I/O 大小总结为直方图：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @bytes = hist(args->bytes); }'

计算块 I/O 请求用户堆栈跟踪：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @\[ustack\] = count(); }'

计算块 I/O 类型标志：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_issue { @\[args->rwbs\] = count(); }'

使用设备和 I/O 类型跟踪块 I/O 错误：

点击这里查看代码图片

bpftrace -e 't:block:block\_rq\_complete /args->error/ {
    printf("dev %d type %s error %d\\n", args->dev, args->rwbs, args->error); }'

计算 SCSI 操作码：

点击这里查看代码图片

bpftrace -e 't:scsi:scsi\_dispatch\_cmd\_start { @opcode\[args->opcode\] =
    count(); }'

计算 SCSI 结果代码：

点击这里查看代码图片

bpftrace -e 't:scsi:scsi\_dispatch\_cmd\_done { @result\[args->result\] = count(); }'

统计 SCSI 驱动程序函数调用：

点击这里查看代码图片

bpftrace -e 'kprobe:scsi\* { @\[func\] = count(); }'

## 联网

通过 PID 和进程名称对套接字接受(2)进行计数：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_accept\* { @\[pid, comm\] = count(); }'

通过 PID 和进程名称对套接字 connect(2) 进行计数：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_connect { @\[pid, comm\] = count(); }'

通过用户堆栈跟踪对套接字 connect(2) 进行计数：

点击这里查看代码图片

bpftrace -e 't:syscalls:sys\_enter\_connect { @\[ustack, comm\] = count(); }'

按方向、CPU 上的 PID 和进程名称对套接字发送/接收进行计数：

点击这里查看代码图片

bpftrace -e 'k:sock\_sendmsg,k:sock\_recvmsg { @\[func, pid, comm\] = count(); }'

通过 CPU 上的 PID 和进程名称来计算套接字发送/接收字节数：

点击这里查看代码图片

bpftrace -e 'kr:sock\_sendmsg,kr:sock\_recvmsg /(int32)retval > 0/ { @\[pid, comm\] =
    sum((int32)retval); }'

通过 CPU 上的 PID 和进程名称计算 TCP 连接数：

点击这里查看代码图片

bpftrace -e 'k:tcp\_v\*\_connect { @\[pid, comm\] = count(); }'

按 CPU 上的 PID 和进程名称计算 TCP 接受数量：

点击这里查看代码图片

bpftrace -e 'k:inet\_csk\_accept { @\[pid, comm\] = count(); }'

通过 CPU 上的 PID 和进程名称对 TCP 发送/接收进行计数：

点击这里查看代码图片

bpftrace -e 'k:tcp\_sendmsg,k:tcp\_recvmsg { @\[func, pid, comm\] = count(); }'

TCP 以直方图形式发送字节：

点击这里查看代码图片

bpftrace -e 'k:tcp\_sendmsg { @send\_bytes = hist(arg2); }'

TCP 接收字节作为直方图：

点击这里查看代码图片

bpftrace -e 'kr:tcp\_recvmsg /retval >= 0/ { @recv\_bytes = hist(retval); }'

按类型和远程主机计算 TCP 重传次数（假设 IPv4）：

点击这里查看代码图片

bpftrace -e 't:tcp:tcp\_retransmit\_\* { @\[probe, ntop(2, args->saddr)\] = count(); }'

统计所有 TCP 函数（给 TCP 增加了很高的开销）：

点击这里查看代码图片

bpftrace -e 'k:tcp\_\* { @\[func\] = count(); }'

通过 CPU 上的 PID 和进程名称来计算 UDP 发送/接收的数量：

点击这里查看代码图片

bpftrace -e 'k:udp\*\_sendmsg,k:udp\*\_recvmsg { @\[func, pid, comm\] = count(); }'

UDP 以直方图形式发送字节：

点击这里查看代码图片

bpftrace -e 'k:udp\_sendmsg { @send\_bytes = hist(arg2); }'

UDP 接收字节作为直方图：

点击这里查看代码图片

bpftrace -e 'kr:udp\_recvmsg /retval >= 0/ { @recv\_bytes = hist(retval); }'

计算传输内核堆栈跟踪：

点击这里查看代码图片

bpftrace -e 't:net:net\_dev\_xmit { @\[kstack\] = count(); }'

显示每个设备的接收 CPU 直方图：

点击这里查看代码图片

bpftrace -e 't:net:netif\_receive\_skb { @\[str(args->name)\] = lhist(cpu, 0, 128, 1); }'

统计ieee80211层函数（给数据包增加很高的开销）：

点击这里查看代码图片

bpftrace -e 'k:ieee80211\_\* { @\[func\] = count()'

计算所有 ixgbevf 设备驱动程序函数（给 ixgbevf 添加高开销）：

点击这里查看代码图片

bpftrace -e 'k:ixgbevf\_\* { @\[func\] = count(); }'

计算所有 iwl 设备驱动程序跟踪点（给 iwl 增加高开销）：

点击这里查看代码图片

bpftrace -e 't:iwlwifi:\*,t:iwlwifi\_io:\* { @\[probe\] = count(); }'