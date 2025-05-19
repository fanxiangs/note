# Ring Buffers

[[null|]][[null|]]As [[null|]][[null|]]discussed in the [kernel documentation](https://oreil.ly/RN_RA), ring buffers are preferred over perf buffers partly for performance reasons, but also to ensure that the ordering of data is preserved, even if the data is submitted by different CPU cores. There is just one buffer, shared across all cores.

There aren’t many changes needed to convert _hello-buffer-config.py_ to use a ring buffer. In the accompanying GitHub repo you’ll find this example as _chapter4/hello-ring-buffer-config.py_. [[#differences_between_example_bcc_code_us|Table 4-2]] shows the differences.

Table 4-2. Differences between example BCC code using a perf buffer and a ring buffer

_hello-buffer-config.py_

_hello-ring-buffer-config.py_

`BPF_PERF_OUTPUT(output);`

`BPF_RINGBUF_OUTPUT(output, 1);`

`output.perf_submit(ctx, &data, sizeof(data));`

`output.ringbuf_output(&data, sizeof(data), 0);`

`b["output"].      open_perf_buffer(print_event)`

`b["output"].      open_ring_buffer(print_event)`

`b.perf_buffer_poll()`

`b.ring_buffer_poll()`

As you’d expect, since these changes relate only to the `output` buffer, the syscalls related to loading the program and the `config` map and attaching the program to the kprobe event all remain unchanged.

The `bpf()` syscall that creates the `output` ring buffer map looks like this:

bpf(BPF\_MAP\_CREATE, {map\_type=BPF\_MAP\_TYPE\_RINGBUF, key\_size=0, value\_size=0,
max\_entries=4096, ... map\_name="output", ...}, 128) = 4

The major difference in the `strace` output is that there is no sign of the series of four different `perf_event_open()`, `ioctl()`, and `bpf(BPF_MAP_UPDATE_ELEM)` system calls that you observed during the setup of a perf buffer. For a ring buffer, there’s just the one file descriptor shared across all CPU cores.

At the time of this writing, BCC is using the `ppoll` mechanism I showed earlier for perf buffers, but it uses the newer `epoll` mechanism to wait for data from the ring buffer. Let’s use this as an opportunity to understand the difference between `ppoll` and `epoll`.

In the perf buffer example, I showed _hello-buffer-config.py_ generating a `ppoll()` syscall, like this:

ppoll(\[{fd=8, events=POLLIN}, {fd=9, events=POLLIN}, {fd=10, events=POLLIN},
{fd=11, events=POLLIN}\], 4, NULL, NULL, 0) = 1 (\[{fd=8, revents=POLLIN}\])

Notice that this passes in the set of file descriptors `8`, `9`, `10`, and `11` from which the user space process wants to retrieve data. Every time this poll event returns data, another call has to be made to `ppoll()` to set up the same set of file descriptors all over again. When using `epoll`, the file descriptor set is managed in a kernel object.

You can see this in the following sequence of `epoll`\-related system calls made when _hello-ring-buffer-config.py_ is setting up access to the `output` ring buffer.

First, the user space program asks for a new `epoll` instance to be created in the kernel:

epoll\_create1(EPOLL\_CLOEXEC) = 8

This returns file descriptor `8`. Then there is a call to `epoll_ctl()`, which tells the kernel to add file descriptor `4` (the `output` buffer) to the set of file descriptors in that `epoll` instance:

epoll\_ctl(8, EPOLL\_CTL\_ADD, 4, {events=EPOLLIN, data={u32=0, u64=0}}) = 0

The user space program uses `epoll_pwait()` to wait until data is available in the ring buffer. This call only returns when data is available:

epoll\_pwait(8,  \[{events=EPOLLIN, data={u32=0, u64=0}}\], 1, -1, NULL, 8) = 1

Of course, if you’re writing code using a framework like BCC (or _libbpf_ or any of the other libraries I’ll describe later in this book), you really don’t need to know these underlying details about how your user space application gets information from the kernel via perf or ring buffers. I hope you’ve found it interesting to get a peek under the covers to see how these things work.

However, you might well find yourself writing code that accesses a map from user space, and it might be helpful to see an example of how this is achieved. Earlier in this chapter, I used `bpftool` to examine the contents of the `config` map. Since it’s a utility that runs in user space, let’s use `strace` to see what syscalls it’s making to retrieve this information.[[null|]][[null|]]