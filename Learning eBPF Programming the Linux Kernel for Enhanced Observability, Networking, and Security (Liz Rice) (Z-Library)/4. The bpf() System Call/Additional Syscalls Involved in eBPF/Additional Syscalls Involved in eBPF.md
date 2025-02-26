# Additional Syscalls Involved in eBPF

To recap, so far you have seen `bpf()` syscalls that add the BTF data, program and maps, and map data to the kernel. The next thing the `strace` output shows relates to setting up the perf buffer.

###### Note

The rest of this chapter dives relatively deeply into the syscall sequences involved when using perf buffers, ring buffers, kprobes, and map iterations. Not all eBPF programs need to do these things, so if you’re in a hurry or you’re finding it a bit too detailed, feel free to skip to the chapter summary. I won’t be offended!