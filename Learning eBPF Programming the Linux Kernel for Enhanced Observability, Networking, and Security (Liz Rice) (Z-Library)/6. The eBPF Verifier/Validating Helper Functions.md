# Validating Helper Functions

[[null|]]You’re not allowed to call directly from eBPF programs to any kernel function (unless it has been registered as a kfunc, which you’ll meet in the next chapter), but eBPF provides a number of helper functions that enable programs to access information from the kernel. There’s a [bpf-helpers manpage](https://oreil.ly/pdLGW) that attempts to document them all.

Different helper functions are valid for different BPF program types. For example, the helper function `bpf_get_current_pid_tgid()` retrieves the current user space process ID and thread ID, but it doesn’t make sense to call this from an XDP program that is triggered by the receipt of a packet at a network interface, because there is no user space process involved. You can see an example of this by changing the `SEC()` definition for the _hello_ eBPF program in _hello-verifier.bpf.c_ from `kprobe` to `xdp`. On attempting to load this program the verifier output gives the following message:

...
16: (85) call bpf\_get\_current\_pid\_tgid#14
unknown func bpf\_get\_current\_pid\_tgid#14

The `unknown func` doesn’t mean the function is completely unknown, just that it is unknown _for this BPF program type_. (BPF program types are a topic for the next chapter; for now you can just think of them as being programs that are suitable for attaching to different types of event.)