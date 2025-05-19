# Summary

In this chapter you saw some example C source code transformed into eBPF bytecode and then compiled to machine code so that it’s ready to be executed in the kernel. You also learned how to use `bpftool` to inspect programs and maps loaded into the kernel, and to attach to XDP events.

In addition, you saw examples of different types of eBPF programs triggered by different kinds of events. An XDP event is triggered by the arrival of a packet of data on a network interface, whereas kprobe and tracepoint events are triggered by hitting some particular point in kernel code. I’ll discuss some other eBPF program types in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].

You also learned how maps are used to implement global variables for eBPF programs, and you saw BPF to BPF function calls.

The next chapter goes into another level of detail as I show you what’s happening at the system call level when `bpftool`—or any other user space code—loads programs and attaches them to events.[[null|]][[null|]]