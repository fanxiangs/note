# Networking

[[null|]][[null|]][[null|]]There are lots of different eBPF program types intended to process network messages as they pass through various points in the network stack. [[#bpf_program_types_hook_into_various_poi|Figure 7-1]] shows where some of the commonly used program types attach. These program types all require `CAP_NET_ADMIN` and `CAP_BPF`, or `CAP_SYS_ADMIN`, capabilities to be permitted.

The context passed to these types of programs is the network message in question, although the type of structure depends on the data the kernel has at the relevant point in the network stack. At the bottom of the stack, data is held in the form of Layer 2 network packets, which are essentially a series of bytes that have been or are ready to be transmitted “on the wire.” At the top of the stack, applications use sockets, and the kernel creates socket buffers to handle data being sent and received from these sockets.

![BPF program types hook into various points in the network stack](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0701.png)

###### Figure 7-1. BPF program types hook into various points in the network stack

###### Note

The network layer model is beyond the scope of this book, but it’s covered in many other books, posts, and training courses. I discussed it in [[ch10.xhtml#ebpf_programming|Chapter 10]] of [Container Security](https://www.oreilly.com/library/view/container-security/9781492056690/) (O’Reilly). For the purposes of this book, it’s sufficient to know that Layer 7 covers formats intended for applications to use, such as HTTP, DNS, or gRPC; TCP is at Layer 4; IP is at Layer 3; and Ethernet and WiFi are at Layer 2. One of the roles of the networking stack is to convert messages between these different formats.

[[null|]]One big difference between the networking program types and the tracing-related types you saw earlier in this chapter is that they are generally intended to allow for the customization of networking behaviors. That involves two main characteristics:

1.  Using a return code from the eBPF program to tell the kernel what to do with a network packet—which could involve processing it as usual, dropping it, or redirecting it to a different destination
    
2.  Allowing the eBPF program to modify network packets, socket configuration parameters, and so on
    

You’ll see some examples of how these characteristics are used to build powerful networking capabilities in the next chapter, but for now, here’s an overview of the eBPF program types.