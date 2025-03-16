# BPF Links

[[null|]][[null|]]BPF links provide a layer of abstraction between an eBPF program and the event it’s attached to. A BPF link itself can be pinned to the filesystem, which creates an additional reference to the program. This means the user space process that loaded the program into the kernel can terminate, leaving the program loaded. The file descriptor for the user space loader program gets freed up, decreasing the count of references to the program, but the reference count will be nonzero because of the BPF link.

You’ll get an opportunity to see BPF links in action if you follow the exercises at the end of this chapter. For now, let’s get back to the sequence of `bpf()` syscalls used by _hello-buffer-config.py_.[[null|]][[null|]][[null|]]