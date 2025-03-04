# Fentry/Fexit

[[null|]][[null|]][[null|]][[null|]]A more efficient mechanism for tracing the entry to and exit from kernel functions was introduced along with the idea of _BPF trampoline_ in kernel version 5.5 (on x86 processors; BPF trampoline support doesn’t arrive for [ARM processors until Linux 6.0](https://oreil.ly/ccuz1)). If you’re using a recent enough kernel, fentry/fexit is now the preferred method for tracing the entry to or exit from a kernel function. You can write the same code inside a kprobe or fentry type program.

There’s an example fentry program called `fentry_execve()` in _chapter7/hello.bpf.c_. I declared the eBPF program for this kprobe using _libbpf_’s macro `BPF_PROG`, which is another convenient wrapper giving access to typed parameters rather than the generic context pointer, but this version is used for fentry, fexit, and tracepoint program types. The definition looks like this:

    SEC

The section name tells _libbpf_ to attach to the fentry hook at the start of the `d⁠o⁠_​e⁠x⁠e⁠c⁠v⁠e⁠(⁠)` kernel function. Just as in the kprobe example, the context parameters reflect the parameters passed to the kernel function where you want to attach this eBPF program.

Fentry and fexit attachment points were designed to be more efficient than kprobes, but there’s another advantage when you want to generate an event at the end of a function: the fexit hook has access to the input parameters to the function, which kretprobe does not. You can see an example of this in [libbpf-bootstrap’s examples](https://oreil.ly/6HDh_). Both _kprobe.bpf.c_ and _fentry.bpf.c_ are equivalent examples that hook into the `do_unlinkat()` kernel function. The eBPF program attached to the kretprobe has the following signature:

    SEC

The `BPF_KRETPROBE` macro expands to make this a kretprobe program on exit from `do_unlinkat()`. The only parameter the eBPF program receives is `ret`, which holds the return value from `do_unlinkat()`. Compare this to the fexit version:

    SEC

In this version the program gets access not just to the return value `ret`, but also to the input parameters to `do_unlinkat()`, which are `dfd` and `name`.