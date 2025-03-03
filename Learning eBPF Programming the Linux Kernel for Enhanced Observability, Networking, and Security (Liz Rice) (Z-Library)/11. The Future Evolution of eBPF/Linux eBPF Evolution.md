# Linux eBPF Evolution

The capabilities of eBPF have evolved with practically every kernel release since 3.15. If you want to know what features are available in any given version, the BCC project maintains a [useful list](https://oreil.ly/4H5hU). And I certainly expect more additions over the coming years.

The best way to predict what’s coming is simply to listen to the people who are working on it. For example, at the 2022 Linux Plumbers Conference, eBPF maintainer Alexei Starovoitov gave a talk discussing how he expects to see the C language used by eBPF programs to evolve.[^4] We’ve already seen eBPF evolve from supporting a few thousand instructions to practically unlimited complexity, with the addition of support for loops and an ever-increasing set of BPF helper functions. As additional capabilities are added into the C that’s supported, and with the support of the verifier, eBPF C could evolve to allow all the flexibility of developing kernel modules, but with the safety and dynamic loading characteristics of eBPF.

Some of the other ideas being discussed and developed for new eBPF features and capabilities include:

Signed eBPF programs

Software supply chain security has been a hot topic for the past few years, and a key element is the ability to check that a program you’re thinking of running comes from the expected source and has not been tampered with. One way to achieve this is, in general, to validate a cryptographic signature that accompanies a program. You might think this is something the kernel could do for eBPF programs, perhaps as part of the verification step, but unfortunately this is not straightforward! As you’ve seen in this book, user space loaders dynamically adjust programs with information about where maps are located, and for CO-RE purposes, which from a signing perspective is hard to distinguish from malicious modifications. This is a problem for which the eBPF community is [keen to find a solution](https://oreil.ly/ns03-).

Long-lived kernel pointers

An eBPF program can retrieve a pointer to a kernel object using a helper function or a kfunc, but a pointer is valid only during that execution of the program. The pointer cannot be stored in a map for later retrieval. The idea of [typed pointer support](https://oreil.ly/fWVdo) will allow for more flexibility in this area.

Memory allocation

It’s not safe for eBPF programs to simply call memory allocation functions like `kmalloc()`, but there is [a proposal that suggests](https://oreil.ly/Yxxc5) an eBPF-specific alternative.

When will you be able to take advantage of new eBPF features as they emerge? As an end user, the features you’re able to take advantage of depend on the version of the kernel you’re running in production, and as I discussed in [[ch01.xhtml#what_is_ebpf_and_why_is_it_importantque|Chapter 1]], it can take several years for kernel releases to make it to stable distributions of Linux. As an individual you might opt for a bleeding-edge kernel, but the vast majority of organizations running server deployments use a stable, supported version. eBPF programmers have to take into account that if they write code that takes advantage of the newest features added to the kernel, the features are unlikely to be usable in most production environments for some years to come. Some organizations will have sufficiently urgent needs that it’s worth rolling out newer kernel versions more quickly in order to early-adopt new eBPF features.

For example, in another forward-looking talk on [building tomorrow’s networking](https://oreil.ly/IvPgd), Daniel Borkmann discussed a feature called _Big TCP_. This was added to Linux in version 5.19 to enable network speeds of 100 GBit/s (and faster) by batching up network packets to be processed in the kernel. Most Linux distributions won’t support a kernel this recent for a few years, but for specialist organizations dealing with large amounts of network traffic, it might well be worth upgrading sooner. Adding Big TCP support into eBPF and Cilium today means it’s available for those massive-scale users, even if it’s not something that can be enabled by most of us for a while.

Since eBPF allows kernel code to be adjusted dynamically, it’s reasonable to expect it to be used to address problems “in the field.” In [[ch09.xhtml#ebpf_for_security|Chapter 9]] you read about using eBPF to mitigate kernel vulnerabilities; work is also underway to use eBPF to help support hardware devices such as [human interface devices](https://oreil.ly/JVYcY) like mice, keyboards, and game controllers. This builds on existing support for decoding the protocols used by infrared controllers that I mentioned in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].