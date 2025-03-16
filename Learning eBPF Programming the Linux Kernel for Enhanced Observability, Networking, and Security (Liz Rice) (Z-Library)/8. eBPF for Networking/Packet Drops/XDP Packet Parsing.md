# XDP Packet Parsing

[[null|]][[null|]]Here’s the definition of the `xdp_md` structure:

    struct

Don’t be fooled by the `__u32` type for the first three fields, as they are really pointers. The `data` field indicates the location in memory where the packet starts, and `data_end` shows where it ends. As you saw in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]], to pass the eBPF verifier you will have to explicitly check that any reads or writes to the packet’s contents are within the range `data` to `data_end`.

There is also an area in memory ahead of the packet, between `data_meta` and `data`, for storing metadata about this packet. This can be used for coordination between multiple eBPF programs that might process the same packet at various places on its journey through the stack.

To illustrate the basics of parsing a network packet, there is an XDP program called `ping()` in the example code, which will simply generate a line of trace whenever it detects a ping (ICMP) packet. Here’s the code for that program:

    SEC

You can see this program in action by following these steps:

1.  Run `make` in the _chapter8_ directory. This doesn’t just build the code; it also attaches the XDP program to the loopback interface (called `lo`).
    
2.  Run `ping localhost` in one terminal window.
    
3.  In another terminal window, watch the output generated in the trace pipe by running `cat /sys/kernel/tracing/trace_pipe`.
    

You should see two lines of trace being generated approximately every second, and they should look like this:

ping-26622   \[000\] d.s11 276880.862408: bpf\_trace\_printk: Hello ping
ping-26622   \[000\] d.s11 276880.862459: bpf\_trace\_printk: Hello ping
ping-26622   \[000\] d.s11 276881.889575: bpf\_trace\_printk: Hello ping
ping-26622   \[000\] d.s11 276881.889676: bpf\_trace\_printk: Hello ping
ping-26622   \[000\] d.s11 276882.910777: bpf\_trace\_printk: Hello ping
ping-26622   \[000\] d.s11 276882.910930: bpf\_trace\_printk: Hello ping

There are two lines of trace per second because the loopback interface is receiving both the ping requests and the ping responses.

You can easily modify this code to drop ping packets by adding a line of code to return `XDP_DROP` when the protocol matches, like this:

    if

If you try this, you’ll see that output resembling the following is only generated in the trace output once per second:

ping-26639   \[002\] d.s11 277050.589356: bpf\_trace\_printk: Hello ping
ping-26639   \[002\] d.s11 277051.615329: bpf\_trace\_printk: Hello ping
ping-26639   \[002\] d.s11 277052.637708: bpf\_trace\_printk: Hello ping

The loopback interface receives a ping request, and the XDP program drops it, so the request never gets far enough through the network stack to elicit a response.

Most of the work in this XDP program is being done in a function called `lookup_protocol()` that determines the Layer 4 protocol type. It’s just an example, not a production-quality implementation of parsing a network packet! But it’s sufficient to give you an idea of how parsing in eBPF works.

The network packet that has been received consists of a string of bytes that are laid out as shown in [[#layout_of_an_ip_network_packetcomma_sta|Figure 8-1]].

![Layout of an IP network packet, starting with an Ethernet header, followed by an IP header, and then the Layer 4 data](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0801.png)

##### Figure 8-1. Layout of an IP network packet, starting with an Ethernet header, followed by an IP header, and then the Layer 4 data

The `lookup_protocol()` function takes the `ctx` structure that holds information about where this network packet is in memory and returns the protocol type that it finds in the IP header. The code is as follows:

    unsigned

[[#code_id_8_1|]]

The local variables `data` and `data_end` point to the start and end of the network packet.

[[#code_id_8_2|]]

The network packet should start with an Ethernet header.

[[#code_id_8_3|]]

But you can’t simply assume this network packet is big enough to hold that Ethernet header! The verifier requires that you check this explicitly.

[[#code_id_8_4|]]

The Ethernet header contains a 2-byte field that tells us the Layer 3 protocol.

[[#code_id_8_5|]]

If the protocol type indicates that it’s an IP packet, the IP header immediately follows the Ethernet header.

[[#code_id_8_6|]]

You can’t just assume there’s enough room for that IP header in the network packet. Again the verifier requires that you check explicitly.

[[#code_id_8_7|]]

The IP header contains the protocol byte the function will return to its caller.

The `bpf_ntohs()` function used by this program ensures that the two bytes are in the order expected on this host. Network protocols are big-endian, but most processors are little-endian, meaning they hold multibyte values in a different order. This function converts (if necessary) from network ordering to host ordering. You should use this function whenever you extract a value from a field in a network packet that’s more than one byte long.

The simple example here shows how just a few lines of eBPF code can have a dramatic impact on networking functionality. It’s not hard to imagine how more complex rules about which packets to pass and which packets to drop could result in the features I described at the start of this section: firewalling, DDoS protection, and packet-of-death vulnerability mitigation. Now let’s consider how even more functionality can be provided given the power to modify network packets within eBPF programs[[null|]][[null|]].[[null|]][[null|]][[null|]][[null|]][[null|]]