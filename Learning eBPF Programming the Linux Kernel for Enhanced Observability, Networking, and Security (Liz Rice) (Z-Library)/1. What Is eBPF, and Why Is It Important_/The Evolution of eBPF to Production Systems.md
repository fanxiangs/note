# The Evolution of eBPF to Production Systems

[[null|]][[null|]]A feature called _kprobes_ (kernel probes) had existed in the Linux kernel since 2005, allowing for traps to be set on almost any instruction in the kernel code. Developers could write kernel modules that attached functions to kprobes for debugging or performance measurement purposes.[^3]

The ability to attach eBPF programs to kprobes was added in 2015, and this was the starting point for a revolution in the way tracing is done across Linux systems. At the same time, hooks started to be added within the kernel’s networking stack, allowing eBPF programs to take care of more aspects of networking functionality. We’ll see more of this in [[ch08.xhtml#ebpf_for_networking|Chapter 8]].

[[null|]]By 2016, eBPF-based tools were being used in production systems. [Brendan Gregg’s](https://www.brendangregg.com) work on tracing at Netflix became widely known in infrastructure and operations circles, as did [his statement](https://oreil.ly/stV6v) that eBPF “brings The request is not authorized because credentials are missing or invalid. to Linux.” [[null|]]In the same year, the Cilium project was announced, being the first networking project to use eBPF to replace the entire datapath in container environments.

[[null|]][[null|]][[null|]]The following year Facebook (now Meta) made [Katran](https://oreil.ly/X-WsL) an open source project. Katran, a layer 4 load balancer, met Facebook’s need for a [highly scalable and fast solution](https://oreil.ly/zl4yX). Every single packet to [Facebook.com](http://Facebook.com) since 2017 has passed through eBPF/XDP.[^4] For me personally, this was the year that ignited my excitement about the possibilities enabled by this technology, after seeing [Thomas Graf’s talk](https://oreil.ly/g9ya0) about eBPF and the [Cilium project](https://oreil.ly/doKbd) at DockerCon in Austin, Texas.

[[null|]][[null|]][[null|]]In 2018, eBPF became a separate subsystem within the Linux kernel, with [Daniel Borkmann](http://borkmann.ch) from Isovalent and [Alexei Starovoitov](https://oreil.ly/K8nXI) from Meta as its maintainers (they were later joined by [Andrii Nakryiko](https://nakryiko.com), also from Meta). [[null|]]The same year saw the introduction of BPF Type Format (BTF), which makes eBPF programs much more portable. We’ll explore this in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]].

[[null|]]The year 2020 saw the introduction of LSM BPF, allowing eBPF programs to be attached to the Linux Security Module (LSM) kernel interface. This indicated that a third major use case for eBPF had been identified: it became clear that eBPF is a great platform for security tooling, in addition to networking and observability.

Over the years, eBPF’s capabilities have grown substantially, thanks to the work of more than 300 kernel developers and many contributors to the associated user space tools (such as `bpftool`, which we’ll meet in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]), compilers, and programming language libraries. [[null|]][[null|]]Programs were once limited to 4,096 instructions, but that limit has grown to 1 million verified instructions[^5] and has effectively been rendered irrelevant by support for tail calls and function calls (which you’ll see in Chapters [^2] and [^3]).

###### Note

For deeper insight into the history of eBPF, who better to refer to than the maintainers who have been working on it from the beginning?

Alexei Starovoitov gave a fascinating presentation about the [history of BPF](https://youtu.be/DAvZH13725I) from its roots in software-defined networking (SDN). In this talk, he discusses the strategies used to get the early eBPF patches accepted into the kernel and reveals that the official birthday of eBPF is September 26, 2014, which marked the acceptance of the first set of patches covering the verifier, BPF system call, and maps.

Daniel Borkmann has also discussed the history of BPF and its evolution to support networking and tracing functionality. I highly recommend his talk [“eBPF and Kubernetes: Little Helper Minions for Scaling Microservices”](https://youtu.be/99jUcLt3rSk), which is full of interesting nuggets of information.[[null|]][[null|]]