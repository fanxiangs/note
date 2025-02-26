# Load Balancing and Forwarding

[[null|]][[null|]][[null|]]XDP programs aren’t limited to inspecting the contents of a packet. They can also modify the packet’s contents. Let’s consider what’s involved if you want to build a simple load balancer that takes packets sent to a given IP address and fans those requests to a number of backends that can fulfill the request.

There’s an example of this in the GitHub repo.[^2] The setup here is a set of containers that run on the same host. There’s a client, a load balancer, and two backends, each running in their own container. As illustrated in [[#example_load_balancer_setup|Figure 8-2]], the load balancer receives traffic from the client and forwards it to one of the two backend containers.

![Example load balancer setup](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0802.png)

###### Figure 8-2. Example load balancer setup

The load balancing function is implemented as an XDP program attached to the load balancer’s eth0 network interface. The return code from this program is `XDP_TX`, indicating that the packet should be sent back out of the interface it came in on. But before that happens, the program has to update the address information in the packet headers.

Although I think it’s useful as a learning exercise, this example code is very, very far from being production ready; for example, it uses hard-coded addresses that assume the exact setup of IP addresses shown in [[#example_load_balancer_setup|Figure 8-2]]. It assumes that the only TCP traffic it will ever receive is requests from the client or responses to the client. It also cheats by taking advantage of the way Docker sets up virtual MAC addresses, using each container’s IP address as the last four bytes of the MAC address for the virtual Ethernet interface for each container. That virtual Ethernet interface is called eth0 from the perspective of the container.

Here’s the XDP program from the example load balancer code:

    SEC

[[#code_id_8_8|]]

The first part of this function is practically the same as in the previous example: it locates the Ethernet header and then the IP header in the packet.

[[#code_id_8_9|]]

This time it will process only TCP packets, passing anything else it receives on up the stack as if nothing had happened.

[[#code_id_8_10|]]

Here the source IP address is checked. If this packet didn’t come from the client, I will assume it is a response going to the client.

[[#code_id_8_11|]]

This code generates a pseudorandom choice between backends A and B.

[[#code_id_8_12|]]

The destination IP and MAC addresses are updated to match whichever backend was chosen…

[[#code_id_8_13|]]

…or if this is a response from a backend (which is the assumption here if it didn’t come from a client), the destination IP and MAC addresses are updated to match the client.

[[#code_id_8_14|]]

Wherever this packet is going, the source addresses need to be updated so that it looks as though the packet originated from the load balancer.

[[#code_id_8_15|]]

The IP header includes a checksum calculated over its contents, and since the source and destination IP addresses have both been updated, the checksum also needs to be recalculated and replaced in this packet.

###### Note

Since this is a book on eBPF and not networking, I haven’t delved into details such as why the IP and MAC addresses need to be updated or what happens if they aren’t. If you’re interested, I cover this some more in my [YouTube video of the eBPF Summit talk](https://oreil.ly/mQxtT) where I originally wrote this example code.

[[null|]]Much like the previous example, the Makefile includes instructions to not only build the code but also use `bpftool` to load and attach the XDP program to the interface, like this:

xdp: $(BPF\_OBJ)
   bpftool net detach xdpgeneric dev eth0
   rm -f /sys/fs/bpf/$(TARGET)
   bpftool prog load $(BPF\_OBJ) /sys/fs/bpf/$(TARGET)
   bpftool net attach xdpgeneric pinned /sys/fs/bpf/$(TARGET) dev eth0

This `make` instruction needs to be run _inside_ the load balancer container so that eth0 corresponds to its virtual Ethernet interface. This leads to an interesting point: an eBPF program is loaded into the kernel, of which there is only one; yet the attachment point may be within a particular network namespace and visible only within that network namespace.[^3][[null|]][[null|]][[null|]]