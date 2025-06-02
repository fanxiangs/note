# Coordinated Network Programs

A [[null|]]complex networking implementation like Cilium can’t be written as a single eBPF program. As shown in [[#cilium_consists_of_multiple_coordinated|Figure 8-7]], it provides several different eBPF programs that are hooked into different parts of the kernel and its network stack.

![Cilium consists of multiple coordinated eBPF programs that hook into different points in the kernel](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0807.png)

##### Figure 8-7. Cilium consists of multiple coordinated eBPF programs that hook into different points in the kernel

As a general principle, Cilium intercepts traffic as soon as it can in order to shorten the processing path for each packet. Messages flowing out from an application pod are intercepted at the socket layer, as close to the application as possible. Inbound packets from the external network are intercepted using XDP. But what about the additional attachment points?

Cilium supports different networking modes that suit different environments. A full description of this is beyond the scope of this book (you can find more information at [Cilium.io](https://cilium.io)), but I’ll give a brief overview here so that you can see why there are so many different eBPF programs!

There is a simple, flat networking mode, in which Cilium allocates IP addresses for all the pods in a cluster from the same CIDR and directly routes traffic between them. There are also a couple of different tunneling modes, in which traffic intended for a pod on a different node gets encapsulated in a message addressed to that destination node’s IP address and decapsulated on that destination node for the final hop into the pod. Different eBPF programs get invoked to handle traffic depending on whether a packet is destined for a local container, the local host, another host on this network, or a tunnel.

[[null|]]In [[#cilium_consists_of_multiple_coordinated|Figure 8-7]] you can see multiple TC programs that handle traffic to and from different devices. These devices represent the possible different real and virtual network interfaces where a packet might be flowing:

*   The interface to a pod’s network (one end of the virtual Ethernet connection between the pod and the host)
    
*   The interface to a network tunnel
    
*   The interface to a physical network device on the host
    
*   The host’s own network interface
    

##### Note

If you’re interested in learning more about how packets flow through Cilium, Arthur Chiao wrote this detailed and interesting blog post: [“Life of a Packet in Cilium: Discovering the Pod-to-Service Traffic Path and BPF Processing Logics”](https://oreil.ly/toxsM).[[null|]]

The different eBPF programs attached at these various points in the kernel communicate using eBFP maps and using the metadata that can be attached to network packets as they flow through the stack (which I mentioned when I discussed accessing network packets in the XDP example). These programs don’t just route packets to their destination; they’re also used to drop packets—just like you saw in earlier examples—based on network policies.