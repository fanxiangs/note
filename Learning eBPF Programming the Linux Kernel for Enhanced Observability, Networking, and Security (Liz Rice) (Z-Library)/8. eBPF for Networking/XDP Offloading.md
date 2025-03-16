# XDP Offloading

[[null|]][[null|]]The idea for XDP originated from a conversation speculating how useful it would be if you could run eBPF programs on a network card to make decisions about individual packets before they even reach the kernel’s networking stack.[^4] There are some network interface cards that support this full _XDP offload_ capability where they can indeed run eBPF programs on inbound packets on their own processor. This is illustrated in [[#network_interface_cards_that_support_xd|Figure 8-3]].

![Network interface cards that support XDP offload can process, drop, and retransmit packets without any work required from the host CPU](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0803.png)

###### Figure 8-3. Network interface cards that support XDP offload can process, drop, and retransmit packets without any work required from the host CPU

This means a packet that gets dropped or redirected back out of the same physical interface—like the packet drop and load balancing examples earlier in this chapter—is never seen by the host’s kernel, and no CPU cycles on the host machine are ever spent processing them, as all the work is done on the network card.

Even if the physical network interface card doesn’t support full XDP offload, many NIC drivers support XDP hooks, which minimizes the memory copying required for an eBPF program to process a packet.[^5]

This can result in significant performance benefits and allows functionality like load balancing to run very efficiently on commodity hardware.[^6]

You’ve seen how XDP can be used to process inbound network packets, accessing them as soon as possible as they arrive on a machine. eBPF can also be used to process traffic at other points in the network stack, in whatever direction it is flowing. Let’s move on and think about eBPF programs attached within the TC subsystem.