# Kprobes and Kretprobes

[[null|]][[null|]]I discussed the concept of kprobes in [[ch01.xhtml#what_is_ebpf_and_why_is_it_importantque|Chapter 1]]. You can attach kprobe programs almost anywhere in the kernel.[^1] Commonly, they are attached using kprobes to the entry to a function and kretprobes to the exit of a function, but you can use kprobes to attach to an instruction that is some specified offset after the entry to the function. If you choose to do this,[^2] you’d need to be confident that the kernel version you’re running on has the instruction you want to attach to where you think it is! Attaching to kernel function entry and exit points can be relatively stable, but arbitrary lines of code might easily be modified from one release to the next.

##### Note

In the example output from `bpftool perf list`, you can see that there is an offset of 0 for both of the kprobes.

When the kernel is compiled, there’s also the possibility that the compiler chooses to “inline” any given kernel function; that is, rather than jump from where the function is called, the compiler might emit the machine code to implement whatever the function does within the calling functions. If a function happens to get inlined, there won’t be a kprobe entry point for your eBPF program to attach to.