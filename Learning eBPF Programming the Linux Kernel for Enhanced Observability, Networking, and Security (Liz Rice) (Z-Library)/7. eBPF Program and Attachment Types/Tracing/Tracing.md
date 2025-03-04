# Tracing

[[null|]]Programs that attach to kprobes, tracepoints, raw tracepoints, fentry/fexit probes, and perf events were all designed to provide an efficient way for eBPF programs in the kernel to report tracing information about events into user space. These tracing-related types weren’t expected to influence the way the kernel behaves in response to the events they are attached to (although, as you’ll see in [[ch09.xhtml#ebpf_for_security|Chapter 9]], there have been some innovations in this area!).

[[null|]][[null|]]These are sometimes referred to as “perf-related” programs. [[null|]]For example, the `bpftool perf` subcommand lets you view programs attached to perf-related events like this:

$ sudo bpftool perf show
pid 232272  fd 16: prog\_id 392  kprobe  func \_\_x64\_sys\_execve  offset 0
pid 232272  fd 17: prog\_id 394  kprobe  func do\_execve  offset 0
pid 232272  fd 19: prog\_id 396  tracepoint  sys\_enter\_execve
pid 232272  fd 20: prog\_id 397  raw\_tracepoint  sched\_process\_exec
pid 232272  fd 21: prog\_id 398  raw\_tracepoint  sched\_process\_exec

The preceding output is what I see when running example code from the _hello.bpf.c_ file in the _chapter7_ directory, which attaches different programs to a variety of events that are all related to `execve()`. I’ll discuss all of these types in this section, but as an overview, these programs are:

*   A kprobe attached to the entry point to the `execve()` system call.
    
*   A kprobe attached to a kernel function, `do_execve()`.
    
*   A tracepoint placed at the entry to the `execve()` syscall.
    
*   Two versions of a raw tracepoint called during the processing of `execve()`. One of these, as you’ll see in this section, is a BTF-enabled version.
    

You’ll need `CAP_PERFMON` and `CAP_BPF` or `CAP_SYS_ADMIN` capabilities to use any of the tracing-related eBPF program types.