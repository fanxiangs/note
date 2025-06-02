# eBPF’s Roots: The Berkeley Packet Filter

[[null|]][[null|]]What we call “eBPF” today has its roots in the BSD Packet Filter, first described in 1993 in a paper[^1] written by Lawrence Berkeley National Laboratory’s [[null|]][[null|]]Steven McCanne and Van Jacobson. This paper discusses a pseudomachine that can run _filters_, which are programs written to determine whether to accept or reject a network packet. These programs were written in the BPF instruction set, a general-purpose set of 32-bit instructions that closely resembles assembly language. Here’s an example taken directly from that paper:

ldh     \[12\] 
jeq     #ETHERTYPE IP, L1, L2 
L1:     ret     #TRUE 
L2:     ret     #0

This tiny piece of code filters out packets that aren’t Internet Protocol (IP) packets. The input to this filter is an Ethernet packet, and the first instruction (`ldh`) loads a 2-byte value starting at byte 12 in this packet. In the next instruction (`jeq`) that value is compared with the value that represents an IP packet. If it matches, execution jumps to the instruction labeled `L1`, and the packet is accepted by returning a nonzero value (identified here as `#TRUE`). If it doesn’t match, the packet is not an IP packet and is rejected by returning `0`.

You can imagine (or, indeed, refer to the paper to find examples of) more complex filter programs that make decisions based on other aspects of the packet. Importantly, the author of the filter can write their own custom programs to be executed within the kernel, and this is the heart of what eBPF enables.

BPF came to stand for “Berkeley Packet Filter,” and it was first introduced to Linux in 1997, in kernel version 2.1.75,[^2] where it was used in the tcpdump utility as an efficient way to capture the packets to be traced out.

[[null|]]Fast-forward to 2012, when seccomp-bpf was introduced in version 3.5 of the kernel. This enabled the use of BPF programs to make decisions about whether to allow or deny user space applications from making system calls. We’ll explore this in more detail in [[ch10.xhtml#ebpf_programming|Chapter 10]]. This was the first step in evolving BPF from the narrow scope of packet filtering to the general-purpose platform it is today. From this point on, the words _packet filter_ in the name started to make less sense!