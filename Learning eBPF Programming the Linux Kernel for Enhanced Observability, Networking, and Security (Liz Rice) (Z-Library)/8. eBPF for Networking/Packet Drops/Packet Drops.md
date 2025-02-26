# Packet Drops

[[null|]][[null|]][[null|]][[null|]][[null|]]There are several network security features that involve dropping certain incoming packets and allowing others. These features include firewalling, DDoS protection, and mitigating packet-of-death vulnerabilities:

*   [[null|]]Firewalling involves deciding on a per-packet basis whether to allow a packet, based on the source and destination IP addresses and/or port numbers.
    
*   [[null|]]DDoS protection adds some complexity, perhaps keeping track of the rate at which packets are arriving from a particular source and/or detecting certain characteristics of the packet contents to determine that an attacker or set of attackers is trying to flood the interface with traffic.
    
*   [[null|]]A packet-of-death vulnerability is a class of kernel vulnerability in which the kernel fails to safely process a packet crafted in a particular way. An attacker who sends packets with this particular format can exploit the vulnerability, which could potentially cause the kernel to crash. Traditionally, when a kernel vulnerability like this is found, it requires installing a new kernel with the fix, which in turn requires machine downtime. But an eBPF program that detects and drops these malicious packets can be installed dynamically, instantly protecting that host without affecting any applications running on the machine.
    

The decision-making algorithms for features like these are beyond the scope of this book, but let’s explore how eBPF programs attached to the XDP hook on a network interface drop certain packets, which is the basis for implementing these use cases.