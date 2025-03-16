# BTF-Enabled Tracepoints

[[null|]]In the previous example I wrote a structure called `my_syscalls_enter_execve` to define the context parameter for my eBPF program. But when you define a structure in your eBPF code or parse the raw arguments, there’s a risk that your code might not match the kernel it’s running on. The good news is that BTF, which you met in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], also solves this problem.

With BTF support, there will be a structure defined in _vmlinux.h_ that matches the context structure passed to a tracepoint eBPF program. Your eBPF program should use the section definition `SEC("tp_btf/_tracepoint name_")` where the tracepoint name is one of the available events listed in _/sys/kernel/tracing/available\_events_. The example program in _chapter7/hello.bpf.c_ looks like this:

    SEC

As you can see, the structure name matches the tracepoint name, prefixed with `trace_event_raw_`.[[null|]][[null|]]