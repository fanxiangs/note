# Setting Up and Reading Perf Events

[[null|]][[null|]][[null|]]I already mentioned that I see four calls to `bpf(BPF_MAP_UPDATE_ELEM)` related to the output perf buffer. With the additional syscalls being traced, the `strace` output shows four sequences, like this:

perf\_event\_open({type=PERF\_TYPE\_SOFTWARE, size=0 /\* PERF\_ATTR\_SIZE\_??? \*/, 
config=PERF\_COUNT\_SW\_BPF\_OUTPUT, ...}, -1, **X**, -1, PERF\_FLAG\_FD\_CLOEXEC) = **Y**

ioctl(**Y**, PERF\_EVENT\_IOC\_ENABLE, 0)      = 0

bpf(BPF\_MAP\_UPDATE\_ELEM, {map\_fd=4, key=0xffffa7842490, value=0xffffa7a2b410,
flags=BPF\_ANY}, 128) = 0

I’ve used `X` to indicate where the output shows values `0`, `1`, `2`, and `3` in the four instances of this call. Referring to the manpage for the `perf_event_open()` syscall, you’ll see that this is the `cpu`, and the field before it is `pid` or process ID. From the manpage:

> pid == -1 and cpu >= 0
> 
> This measures all processes/threads on the specified CPU.

The fact that this sequence happens four times corresponds to there being four CPU cores in my laptop. This, at last, is the explanation for why there are four entries in the “output” perf buffer map: there is one for each CPU core. It also explains the “array” part of the map type name `BPF_MAP_TYPE_PERF_EVENT_ARRAY`, as the map doesn’t just represent one perf ring buffer but an array of buffers, one for each core.

If you write eBPF programs, you won’t need to worry about details like handling the number of cores, as this will be taken care of for you by any of the eBPF libraries discussed in [[ch10.xhtml#ebpf_programming|Chapter 10]], but I think it’s an interesting aspect of the syscalls you see when you use `strace` on this program.

The `perf_event_open()` calls each return a file descriptor, which I’ve represented as `Y`; these have the values `8`, `9`, `10`, and `11`. The `ioctl()` syscalls enable the perf output for each of these file descriptors[[null|]]. The `BPF_MAP_UPDATE_ELEM bpf()` syscalls set the map entry to point to the perf ring buffer for each CPU core to indicate where it can submit data.

User space code can then use `ppoll()` on all four of these output stream file descriptors so that it can get the data output, whichever core happens to run the eBPF program _hello_ for any given `execve()` kprobe event. Here’s the syscall to `ppoll()`:

ppoll(\[{fd=8, events=POLLIN}, {fd=9, events=POLLIN}, {fd=10, events=POLLIN},
{fd=11, events=POLLIN}\], 4, NULL, NULL, 0) = 1 (\[{fd=8, revents=POLLIN}\])

As you’ll see if you try running the example program yourself, these `ppoll()` calls block until there is something to read from one of the file descriptors. You won’t see the return code written to the screen until something triggers `execve()`, which causes the eBPF program to write data that user space retrieves using this `ppoll()` call.

In [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]] I mentioned that if you have a kernel of version 5.8 or above, BPF ring buffers are now preferred over perf buffers.[^4] Let’s take a look at a modified version of the same example code that uses a ring buffer.