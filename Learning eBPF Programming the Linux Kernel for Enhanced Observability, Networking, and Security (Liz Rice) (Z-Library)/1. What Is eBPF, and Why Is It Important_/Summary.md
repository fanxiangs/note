# Summary

I hope this chapter has given you some insight into why eBPF as a platform is so powerful. It allows us to change the behavior of the kernel, providing us the flexibility to build bespoke tools or customized policies. eBPF-based tools can observe any event across the kernel, and hence across all applications running on a (virtual) machine, whether they are containerized or not. eBPF programs can also be deployed dynamically, allowing behavior to be changed on the fly.

So far we’ve discussed eBPF at a relatively conceptual level. In the next chapter we’ll make it more concrete and explore the constituent parts of an eBPF-based application.[[null|]][[null|]][[null|]][[null|]][[null|]][[null|]][[null|]][[null|]][[null|]][[null|]]

[^1] [“The BSD Packet Filter: A New Architecture for User-level Packet Capture”](https://oreil.ly/4GpgQ) by Steven McCanne and Van Jacobson.

[^2] These and other details come from Alexei Starovoitov’s 2015 NetDev presentation, [“BPF – in-kernel virtual machine”](https://oreil.ly/hISe1).

[^3] There is a good description of how kprobes work in [the kernel documentation](https://oreil.ly/Ue6Ii).

[^4] This wonderful fact comes from Daniel Borkmann’s KubeCon 2020 talk titled [“eBPF and Kubernetes: Little Helper Minions for Scaling Microservices”](https://oreil.ly/tIR9o).

[^5] For more details on the instruction limit and “complexity limit,” see [https://oreil.ly/0iVer](https://oreil.ly/0iVer).

[^6] Extract from “What Is eBPF?” by Liz Rice. Copyright © 2022 O’Reilly Media. Used with permission.

[^7] [“Linux 5.12 Coming In At Around 28.8 Million Lines”](https://oreil.ly/9zJP2). Phoronix, March 2021.

[^8] Jiang Y, Adams B, German DM. 2013. [“Will My Patch Make It? And How Fast?”](https://oreil.ly/rj2P4) (2013). According to this research paper, 33% of patches are accepted, and most take three to six months.

[^9] Thankfully, security patches to existing functionality are made available more quickly.[[null|]][[null|]]

[^10] Høiland-Jørgensen T, Brouer JD, Borkmann D, et al. [“The eXpress data path: fast programmable packet processing in the operating system kernel”](https://oreil.ly/qyhLK). _Proceedings of the 14th International Conference on emerging Networking EXperiments and Technologies_ (CoNEXT ’18). Association for Computing Machinery; 2018:54–66.