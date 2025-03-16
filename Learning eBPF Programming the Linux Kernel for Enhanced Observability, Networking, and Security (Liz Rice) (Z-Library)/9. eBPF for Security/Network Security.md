# Network Security

[[null|]][[null|]][[null|]][[ch08.xhtml#ebpf_for_networking|Chapter 8]] discussed how eBPF can be used very effectively to implement network security mechanisms. To summarize:

*   Firewalling and DDoS protection are a natural fit for eBPF programs attached early in the ingress path for network packets. And with the possibility of XDP programs offloaded to hardware, malicious packets may never even reach the CPU!
    
*   For implementing more sophisticated network policies, such as Kubernetes policies determining which services are allowed to communicate with one another, eBPF programs that attach to points in the network stack can drop packets if they are determined to be out of policy.
    

Network security tools are very often used in a preventative mode, dropping packets rather than just auditing malicious activity. This is because it’s so easy for bad actors to mount network-related attacks; if you give a device a public IP address exposed to the internet, it won’t be long before you start seeing suspicious traffic, so organizations are forced to use preventative measures.

In contrast, lots of organizations use intrusion detection tools in an audit mode, and they rely on forensics to determine whether a suspicious event was really malicious and what remedial action needs to be taken. If a given security tool is too blunt an instrument and is prone to detecting false-positives, it’s not surprising that it needs to be run in audit mode rather than preventative mode. It’s my belief that eBPF is enabling more sophisticated security tools with finer-grained, accurate controls. Just as we consider firewalls today to be sufficiently accurate to use in preventative mode, we’ll see increased use of preventative tooling that acts on other, non-networking events. This could even include eBPF-based controls being packaged as part of an application product so that it can provide its own runtime security.