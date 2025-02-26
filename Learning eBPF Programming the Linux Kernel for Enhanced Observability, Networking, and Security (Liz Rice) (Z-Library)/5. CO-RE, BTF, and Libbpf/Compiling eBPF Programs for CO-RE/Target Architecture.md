# Target Architecture

[[null|]][[null|]]If you’re using certain macros defined by _libbpf_, you’ll need to specify the target architecture at compile time. The _libbpf_ header file _bpf/bpf\_tracing.h_ defines several macros that are platform specific, such as `BPF_KPROBE` and `BPF_KPROBE_SYSCALL` that I’ve used in this example. The `BPF_KPROBE` macro can be used for eBPF programs that are being attached to kprobes, and `BPF_KPROBE_SYSCALL` is a variant specifically for syscall kprobes.

The argument to a kprobe is a `pt_regs` structure that holds a copy of the contents of the CPU registers. Since registers are architecture specific, the `pt_regs` structure definition depends on the architecture you’re running on. This means that if you want to use these macros, you’ll need to also tell the compiler what the target architecture is. You can do this by setting `-D __TARGET_ARCH_($ARCH)` where `$ARCH` is an architecture name like arm64, amd64, and so on.

Also note that if you didn’t use the macro, you’d need architecture-specific code to access the register information anyway for a kprobe.

Perhaps “compile once _per architecture_, run everywhere” would have been a bit of a mouthful!