# Exercises

The example code for this chapter includes kprobe, fentry, tracepoint, raw tracepoint, and BTF-enabled tracepoint programs that are all attached to the entry to the same system call. As you know, eBPF tracing programs can be attached to many other places besides syscalls.

1.  Run the example code using `strace` to capture the `bpf()` system calls, like this:
    
    strace -e bpf -o outfile ./hello
    
    This will record information about each `bpf()` syscall into a file called _outfile_. Look for the `BPF_PROG_LOAD` instructions in that file, and see how the `prog_type` file varies for different programs. You can identify which program is which by the `prog_name` field in the trace, and match them to the source code in _chapter7/hello.bpf.c_.
    
2.  The example user space code in _hello.c_ loads all the program objects defined in `hello.bpf.o`. As an exercise in writing _libbpf_ user space code, modify the example code load and attach just one of the eBPF programs (pick whichever one you like), without removing those programs from _hello.bpf.c_.
    
3.  Write a kprobe and/or fentry program that is triggered when some other kernel function is called. You can find the available functions in your kernel version by looking at _/proc/kallsyms_.
    
4.  Write a regular, raw or BTF-enabled tracepoint program that attaches to some other kernel tracepoint. You can find the available tracepoints in `/sys/kernel/tracing/available_events`.
    
5.  Try to attach more than one XDP program to a given interface, and confirm that you can’t! You should see an error that looks something like this:
    
    libbpf: Kernel error message: XDP program already attached
    Error: interface xdpgeneric attach failed: Device or resource busy
    

[^1] Except for a few parts of the kernel where kprobes aren’t permitted for security reasons. These are listed in `/sys/kernel/debug/kprobes/blacklist`.

[^2] The only example I have seen so far is in the [cilium/ebpf test suite](https://oreil.ly/rL5E8).

[^3] Up to Go version 1.17, when a new register-based calling convention was introduced. Nevertheless, I think there will be Go executables built with older versions circulating for some time to come.