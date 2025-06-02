# Traffic Control (TC)

[[null|]][[null|]]I mentioned traffic control in the previous chapter. By the time a network packet reaches this point it will be in kernel memory in the form of an [sk_buff](https://oreil.ly/TKDCF). This is a data structure that’s used throughout the kernel’s network stack. eBPF programs attached within the TC subsystem receive a pointer to the `sk_buff` structure as the context parameter.

###### Note

You might be wondering why XDP programs don’t also use this same structure for their context. The answer is that the XDP hook happens before the network data reaches the network stack and before the `sk_buff` structure has been set up.

The TC subsystem is intended to regulate how network traffic is scheduled. For example, you might want to limit the bandwidth available to each application so that they all get a fair chance. But when you’re looking at scheduling individual packets, _bandwidth_ isn’t a terribly meaningful term, as it’s used for the average amount of data being sent or received. A given application might be very bursty, or another application might be very sensitive to network latency, so TC gives much finer control over the way packets are handled and prioritized.[^7]

eBPF programs were introduced here to give custom control over the algorithms used within TC. But with the power to manipulate, drop, or redirect packets, eBPF programs attached within TC can also be used as the building blocks for complex network behaviors.

A given piece of network data in the stack flows in one of two directions: _ingress_ (inbound from the network interface) or _egress_ (outbound toward the network interface). eBPF programs can be attached in either direction and will affect traffic only in that direction. Unlike XDP, it’s possible to attach multiple eBPF programs that will be processed in sequence.

Traditional traffic control is split into _classifiers_, which classify packets based on some rule, and separate _actions_, which are taken based on the output from a classifier and determine what to do with a packet. There can be a series of classifiers, all defined as part of a _qdisc_ or queuing discipline.

eBPF programs are attached as a classifier, but they can also determine what action to take within the same program. The action is indicated by the program’s return code (whose values are defined in _linux/pkt\_cls.h_):

*   `TC_ACT_SHOT` tells the kernel to drop the packet.
    
*   `TC_ACT_UNSPEC` behaves as if the eBPF program hadn’t been run on this packet (so it would be passed to the next classifier in the sequence, if there is one).
    
*   `TC_ACT_OK` tells the kernel to pass the packet to the next layer in the stack.
    
*   `TC_ACT_REDIRECT` sends the packet to the ingress or egress path of a different network device.
    

Let’s take a look at a few simple examples of programs that can be attached within TC. The first simply generates a line of trace and then tells the kernel to drop the packet:

    int

Now let’s consider how to drop only a subset of packets. This example drops ICMP (ping) request packets and is very similar to the XDP example you saw earlier in this chapter:

    int

The `sk_buff` structure has pointers to the start and end of the packet data, very much like the `xdp_md` structure, and packet parsing proceeds in very much the same way. Again, to pass verification you have to explicitly check that any access to data is within the range between `data` and `data_end`.

You might be wondering why you would want to implement something like this at the TC layer when you have already seen the same kind of functionality implemented with XDP. One good reason is that you can use TC programs for egress traffic, where XDP can only process ingress traffic. Another is that because XDP is triggered as soon as the packet arrives, there is no `sk_buff` kernel data structure related to the packet at that point. If the eBPF program is interested in or wants to manipulate the `sk_buff` the kernel creates for this packet, the TC attachment point is suitable.

###### Note

To better understand the differences between XDP and TC eBPF programs, read the “Program Types” section in the [BPF and XDP Reference Guide](https://oreil.ly/MWAJL) from the Cilium project.

Now let’s consider an example that doesn’t just drop certain packets. This example identifies a ping request being received and responds with a ping response:

    int

[[#code_id_8_16|]]

The `is_icmp_ping_request()` function parses the packet and checks not only that it’s an ICMP message, but also that it’s an echo (ping) request.

[[#code_id_8_17|]]

Since this function is going to send a response to the sender, the source and destination addresses need to be swapped. (You can read the example code if you want to see the nitty-gritty details of this, which also includes updating the IP header checksum.)

[[#code_id_8_18|]]

This is converted to an echo response by changing the type field in the ICMP header.

[[#code_id_8_19|]]

This helper function sends a clone of the packet back through the interface (`skb->ifindex`) on which it was received.

[[#code_id_8_20|]]

Since the helper function cloned the packet before sending out the response, the original packet should be dropped.

In normal circumstances, a ping request would be handled later by the kernel’s network stack, but this small example demonstrates how network functionality more generally can be replaced by an eBPF implementation.

Lots of networking capabilities today are handled by user space services, but where they can be replaced by eBPF programs, it’s likely to be great for performance. A packet that’s processed within the kernel doesn’t have to complete its journey through the rest of the stack; there is no need for it to transition to user space for processing, and the response doesn’t require a transition back into the kernel. What’s more, the two could run in parallel—an eBPF program can return `TC_ACT_OK` for any packet that requires complex processing that it can’t handle so that it gets passed up to the user space service as normal.

For me, this is an important aspect of implementing network functionality in eBPF. As the eBPF platform develops (e.g., more recent kernels allowing programs of one million instructions), it’s possible to implement increasingly complex aspects of networking in the kernel. The parts that are not yet implemented in eBPF can still be handled either by the traditional stack within the kernel or in user space. Over time, more and more features can be moved from user space into the kernel, with the flexibility and dynamic nature of eBPF meaning you won’t have to wait for them to be part of the kernel distribution itself. You can load eBPF implementations immediately, just as I discussed in [[ch01.xhtml#what_is_ebpf_and_why_is_it_importantque|Chapter 1]].

I’ll return to the implementation of networking features in [[#ebpf_and_kubernetes_networking|“eBPF and Kubernetes Networking”]]. But first, let’s consider another use case that eBPF enables: inspecting the decrypted contents of encrypted traffic.[[null|]][[null|]]