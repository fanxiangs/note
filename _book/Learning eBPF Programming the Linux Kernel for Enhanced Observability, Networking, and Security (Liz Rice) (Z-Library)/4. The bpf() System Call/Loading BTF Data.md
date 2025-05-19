# Loading BTF Data

[[null|]][[null|]]The first call to `bpf()` that I see looks like this:

bpf(BPF\_BTF\_LOAD, {btf="\\237\\353\\1\\0...}, 128) = 3

In this case the command you can see in the output is `BPF_BTF_LOAD`. This is just one of a set of valid commands that are (at least at the time of this writing) most comprehensively documented within the kernel source code.[^1]

It’s possible that you won’t see a call with this command if you’re using a relatively old Linux kernel, as it relates to BTF, or BPF Type Format.[^2] BTF allows eBPF programs to be portable across different kernel versions so that you can compile a program on one machine and use it on another, which might be using a different kernel version and hence have different kernel data structures. I’ll discuss this in more detail in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]].

This call to `bpf()` is loading a blob of BTF data into the kernel, and the return code from the `bpf()` system call (`3` in my example) is a file descriptor that refers to that data.

###### Note

[[null|]]A _file descriptor_ is an identifier for an open file (or file-like object). If you open a file (with the `open()` or `openat()` system call) the return code is a file descriptor, which is then passed as an argument to other syscalls such as `read()` or `write()` to perform operations on that file. Here the blob of data isn’t exactly a file, but it is given a file descriptor as an identifier that can be used for future operations that refer to it.