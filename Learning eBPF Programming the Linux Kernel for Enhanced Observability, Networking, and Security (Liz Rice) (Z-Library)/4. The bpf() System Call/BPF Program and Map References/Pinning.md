# Pinning

[[null|]][[null|]]You already saw pinning in action in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]], with the following command:

bpftool prog load hello.bpf.o /sys/fs/bpf/hello

##### Note

These pinned objects aren’t real files persisted to disk. [[null|]]They are created on a _pseudo filesystem_, which behaves like a regular disk-based filesystem with directories and files. But they are held in memory, which means they will not remain in place over a system reboot.

[[null|]]If `bpftool` were to allow you to load the program without pinning it, that would be pointless, because the file descriptor gets released when `bpftool` exits, and if there are zero references, the program would get deleted, so nothing useful would have been achieved. But pinning it to the filesystem means there is an additional reference to the program, so the program remains loaded after the command completes.

The reference counter also gets incremented when a BPF program is attached to a hook that will trigger it. The behavior of these reference counts depends on the BPF program type. You’ll learn more about these program types in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]], but there are some that relate to tracing (like kprobes and tracepoints) and are always associated with a user space process; for these types of eBPF programs, the kernel’s reference count gets decremented when that process exits. Programs that are attached within the network stack or cgroups (short for “control groups”) aren’t associated with any user space process, so they stay in place even after the user space program that loads them exits. [[null|]]You already saw an example of this when loading an XDP program with the `ip link` command:

ip link set dev eth0 xdp obj hello.bpf.o sec xdp

The `ip` command has completed, and there is no definition of a pinned location, but nevertheless, `bpftool` will show you that the XDP program is loaded in the kernel:

$ bpftool prog list
… 
1255: xdp  name hello  tag 9d0e949f89f1a82c  gpl
        loaded\_at 2022-11-01T19:21:14+0000  uid 0
        xlated 48B  jited 108B  memlock 4096B  map\_ids 612

The reference count for this program is nonzero, because of the attachment to the XDP hook that persisted after the `ip link` command completed.

eBPF maps also have reference counters, and they get cleaned up when their reference count drops to zero. Each eBPF program that uses a map increments the counter, as does each file descriptor that user space programs might hold to the map.

It’s possible that the source code for an eBPF program might define a map that the program doesn’t actually reference. Suppose you wanted to store some metadata about a program; you could define it as a global variable, and as you saw in the previous chapter, this information gets stored in a map. If the eBPF program doesn’t do anything with that map, there won’t automatically be a reference count from the program to the map. There’s a `BPF(BPF_PROG_BIND_MAP)` syscall that associates a map with a program so that the map doesn’t get cleaned up as soon as the user space loader program exits and is no longer holding a file descriptor reference to the map.

Maps can also be pinned to the filesystem, and user space programs can gain access to the map by knowing the path to the map.

##### Note

Alexei Starovoitov wrote a good description of BPF reference counters and file descriptors in his blog post [“Lifetime of BPF Objects”](https://oreil.ly/vofxH).

Another way to create a reference to a BPF program is with a BPF link.