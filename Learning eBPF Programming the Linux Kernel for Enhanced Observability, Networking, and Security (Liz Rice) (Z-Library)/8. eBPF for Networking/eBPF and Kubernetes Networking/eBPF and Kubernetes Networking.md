# eBPF and Kubernetes Networking

[[null|]][[null|]]Although this book isn’t about Kubernetes, eBPF is so widely used for Kubernetes networking that it’s a great illustration of using the platform to customize the networking stack.

[[null|]]In Kubernetes environments, applications are deployed in _pods_. Each pod is a group of one or more containers that share kernel namespaces and cgroups, isolating pods from each other and from the host machine they are running on.

In particular (for the purposes of this chapter), a pod typically has its own network namespace and its own IP address.[^9] This means the kernel has a set of network stack structures for that namespace, separated from the host’s and from other pods. As shown in [[#network_path_in_kubernetes|Figure 8-5]], the pod is connected to the host by a virtual Ethernet connection, and it is allocated its own IP address.

![Network path in Kubernetes](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0805.png)

###### Figure 8-5. Network path in Kubernetes

You can see from [[#network_path_in_kubernetes|Figure 8-5]] that a packet coming from outside the machine destined for an application pod has to travel through the network stack on the host, across the virtual Ethernet connection, and into the pod’s network namespace, and then it has to traverse the network stack again to reach the application.

Those two network stacks are running in the same kernel, so the packet is really running through the same processing twice. The more code a network packet has to pass through, the higher the latency, so if it’s possible to shorten the network path, that will likely bring about performance improvements.

An eBPF-based networking solution like Cilium can hook into the network stack to override the kernel’s native networking behavior, as shown in [[#bypassing_iptables_and_conntrack_proces|Figure 8-6]].

![Bypassing iptables and conntrack processing with eBPF](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0806.png)

###### Figure 8-6. Bypassing iptables and conntrack processing with eBPF

In particular, eBPF enables replacing iptables and conntrack with a more efficient solution for managing network rules and connection tracking. Let’s discuss why this results in a significant performance improvement in Kubernetes.