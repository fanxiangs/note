# Helper Functions and Return Codes

[[null|]][[null|]][[null|]]As you saw in the previous chapter, the verifier checks that all helper functions used by a program are compatible with its program type. The example in the previous chapter demonstrated that the `bpf_get_current_pid_tgid()` helper function isn’t permitted in an XDP program. There is no user space process or thread involved at the point where a packet is received and the XDP hook is triggered, so a call to discover the current process and thread ID doesn’t make sense in that context.

[[null|]]The program type also determines the meaning of the return code from the program. [[null|]]Again using XDP as an example, the return code value tells the kernel what to do with the packet once the eBPF program has finished processing it—which could involve passing it to the network stack, dropping it, or redirecting it to a different interface. These return codes wouldn’t make any sense when an eBPF program is triggered by, say, hitting a particular tracepoint, where there is no network packet involved.

There is a [manpage for helper functions](https://oreil.ly/e8K73) (with, quite reasonably, disclaimers that it might not be complete due to the ongoing development of the BPF subsystem).

[[null|]]You can get a list of which helper functions are available for each program type in your version of the kernel with the `bpftool feature` command. This shows the system configuration and lists all the available program types and map types, and even lists all the helper functions that are supported for each program type.

Helper functions are considered part of the _UAPI_, the Linux kernel’s external, stable interface. As such, once a helper function has been defined in the kernel, it shouldn’t change in the future, even though the kernel’s internal functions and data structures can change.

Despite the risk of changes between kernel versions, there was demand from eBPF programmers to be able to access some internal functions from eBPF programs. This can be achieved using the mechanism called _BPF kernel functions_, or [kfuncs](https://oreil.ly/gKSEx)_._