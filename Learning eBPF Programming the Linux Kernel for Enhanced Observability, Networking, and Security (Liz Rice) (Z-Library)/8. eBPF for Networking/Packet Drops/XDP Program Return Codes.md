# XDP Program Return Codes

[[null|]][[null|]][[null|]]An XDP program is triggered by the arrival of a network packet. The program examines the packet, and when it’s done, the return code gives a _verdict_ that indicates what to do next with that packet:

*   `XDP_PASS` indicates that the packet should be sent to the network stack in the normal way (as it would have done if there were no XDP program).
    
*   `XDP_DROP` causes the packet to be discarded immediately.
    
*   `XDP_TX` sends the packet back out of the same interface it arrived on.
    
*   `XDP_REDIRECT` is used to send it to a different network interface.
    
*   `XDP_ABORTED` results in the packet being dropped, but its use implies an error case or something unexpected, rather than a “normal” decision to discard a packet.
    

For some use cases (like firewalling), the XDP program simply has to decide between passing the packet on or dropping it. An outline for an XDP program that decides whether to drop packets looks something like this:

    SEC

An XDP program can also manipulate the packet contents, but I’ll come to that later in this chapter.

XDP programs get triggered whenever an inbound network packet arrives on the interface to which it is attached. The `ctx` parameter is a pointer to an `xdp_md` structure, which holds metadata about the incoming packet. Let’s see how you can use this structure to examine the packet’s contents in order to reach a verdict.