# Exercises and Further Reading

Here are some ways to learn more about the range of networking use cases for eBPF:

1.  Modify the example XDP program `ping()` so that it generates different trace messages for ping responses and ping requests. The ICMP header immediately follows the IP header in the network packet (just like the IP header follows the Ethernet header). You’ll likely want to use `struct icmphdr` from _linux/icmp.h_ and look at whether the type field shows `ICMP_ECHO` or `ICMP_ECHOREPLY`.
    
2.  If you want to dive further into XDP programming, I recommend the xdp-project’s [xdp-tutorial](https://oreil.ly/UmJMF).
    
3.  Use [sslsniff](https://oreil.ly/Zuww7) from the BCC project to view the contents of encrypted traffic.
    
4.  Explore Cilium by using tutorials and labs linked to from the [Cilium website](https://cilium.io/get-started).
    
5.  Use the editor at [networkpolicy.io](https://networkpolicy.io) to visualize the effect of network policies in a Kubernetes deployment.
    

[^1] At the time of this writing, around 100 organizations have publicly announced their use of Cilium in its [USERS.md file](https://oreil.ly/PC7-G), though this number is growing quickly. Cilium has also been adopted by AWS, Google, and Microsoft.

[^2] This example is based on a talk I gave at eBPF Summit 2021 called [“A Load Balancer from scratch”](https://oreil.ly/mQxtT). Build an eBPF load balancer in just over 15 minutes!

[^3] If you want to explore this, try [CTF Challenge 3 from eBPF Summit 2022](https://oreil.ly/YIh_t). I won’t give spoilers here in the book, but you can see the solution in [a walkthrough given by Duffie Cooley and me here](https://oreil.ly/_51rC).

[^4] See Daniel Borkmann’s presentation [“Little Helper Minions for Scaling Microservices”](https://oreil.ly/_8ZuF) that includes a history of eBPF, where he tells this anecdote.

[^5] Cilium maintains a [list of drivers that support XDP](https://oreil.ly/wCMjB) within the [BPF and XDP Reference Guide](https://oreil.ly/eB7vL).

[^6] Ceznam shared data about the performance boost its team saw when experimenting with an eBPF-based load balancer in [this blog post](https://oreil.ly/0cbCx).

[^7] For a more complete overview of TC and its concepts, I recommend Quentin Monnet’s post [“Understanding tc “direct action” mode for BPF”](https://oreil.ly/7gU2A).

[^8] There is also a blog post that accompanies this example at [https://blog.px.dev/ebpf-openssl-tracing](https://blog.px.dev/ebpf-openssl-tracing).

[^9] It’s possible for pods to be run in the host’s network namespace so that they share the IP address of the host, but this isn’t usually done unless there’s a good reason for an application running in the pod to require it.