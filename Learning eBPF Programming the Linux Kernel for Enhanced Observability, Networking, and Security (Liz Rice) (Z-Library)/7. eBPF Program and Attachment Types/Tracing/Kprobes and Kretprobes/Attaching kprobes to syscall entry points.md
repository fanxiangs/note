# Attaching kprobes to syscall entry points

[[null|]][[null|]]The first example eBPF program for this chapter is called `kprobe_sys_execve`, and it is a kprobe attached to the `execve()` syscall. The function and its section definition look like this:

    SEC

This is the same as what you saw in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]].

One reason to attach to syscalls is that they are stable interfaces that won’t change between kernel versions (the same is true of tracepoints, which we’ll come to shortly). However, syscall kprobes shouldn’t be relied on for security tooling, for reasons I’ll cover in detail in [[ch09.xhtml#ebpf_for_security|Chapter 9]].