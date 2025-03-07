# LSM

[[null|]][[null|]]`BPF_PROG_TYPE_LSM` programs are attached to the _Linux Security Module (LSM) API_, which is a stable interface within the kernel originally intended for kernel modules to use to enforce security policies. As you’ll see in [[ch09.xhtml#ebpf_for_security|Chapter 9]], where I’ll discuss this in more detail, eBPF security tooling can now use this interface too.

`BPF_PROG_TYPE_LSM` programs are attached using `bpf(BPF_RAW_TRACEPOINT_OPEN)`, and in many ways they are treated like tracing programs. One interesting characteristic of `BPF_PROG_TYPE_LSM` programs is that the return value affects the way the kernel behaves. A nonzero return code indicates that the security check wasn’t passed, so the kernel won’t proceed with whatever operation it was asked to complete. This is a significant difference from perf-related program types where the return code is ignored.

##### Note

The Linux kernel documentation covers [LSM BPF programs](https://oreil.ly/vcPHY).

The LSM program type isn’t the only one with a role to play in security. Many of the networking-related program types that you’ll see in the next section can be used for network security to permit or deny networking traffic or networking-related operations. You’ll also see more about eBPF being used for security purposes in [[ch09.xhtml#ebpf_for_security|Chapter 9]].

So far in this chapter you have seen how a set of kernel and user space tracing program types enable visibility over the whole system. The next set of eBPF program types to consider are those that let us hook into the network stack, with the option not merely to observe but also to affect how it handles data being sent and received.[[null|]][[null|]]