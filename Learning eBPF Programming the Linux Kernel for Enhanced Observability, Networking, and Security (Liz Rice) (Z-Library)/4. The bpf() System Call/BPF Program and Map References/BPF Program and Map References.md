# BPF Program and Map References

[[null|]][[null|]][[null|]]You know that loading a BPF program into the kernel with the `bpf()` syscall returns a file descriptor. Within the kernel, this file descriptor is a _reference_ to the program. The user space process that made the syscall owns this file descriptor; when that process exits, the file descriptor gets released, and the reference count to the program is decremented. [[null|]]When there are no references left to a BPF program, the kernel removes the program.

An additional reference is created when you _pin_ a program to the filesystem.