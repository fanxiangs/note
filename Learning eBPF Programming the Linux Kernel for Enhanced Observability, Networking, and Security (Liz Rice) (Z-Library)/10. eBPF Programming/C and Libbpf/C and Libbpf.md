# C and Libbpf

[[null|]][[null|]][[null|]]You’ve seen lots of examples in this book of eBPF programs written in C, using the LLVM toolchain to compile into eBPF bytecode. You’ve also seen that extensions were added to support BTF and CO-RE. Many C programmers will also be familiar with the other major C compiler, GCC, and will be happy to hear that [GCC from version 10](https://oreil.ly/XAzxP) also supports compiling to eBPF as a target; however, there are still some gaps compared with the functionality provided in LLVM.

As you saw in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], CO-RE and _libbpf_ enabled an approach to portable eBPF programming that doesn’t require shipping a compiler toolchain alongside every eBPF tool. The BCC project took advantage of this and, in addition to the original set of BCC performance tracing tools, it now has versions of these tools rewritten to take advantage of _libbpf_. The general consensus is that the versions of the BCC tools that have been rewritten based on _libbpf_ are the better option to use, since they have a significantly lower memory footprint[^3] and don’t involve a start-up delay while the compilation step takes place.

If you’re comfortable with programming in C, using _libbpf_ can make a lot of sense. You’ve seen lots of examples of this already in this book.

###### Note

To write your own _libbpf_ programs in C, the best place to start (now that you have read this book!) is [libbpf-bootstrap](https://oreil.ly/4mx81). Read Andrii Nakryiko’s [blog post about it](https://oreil.ly/-OW8v) as a good introduction to the motivations behind this project.

There’s also a library called [libxdp](https://oreil.ly/374mL) that builds on _libbpf_ to allow for easier development and management of XDP programs. This is part of xdp-tools, which also holds one of my favorite learning resources for eBPF programming: the [XDP Tutorial](https://oreil.ly/E6dvl).[^4]

But C is quite a challenging, low-level language. C programmers have to take responsibility for things like memory management and buffer handling, and it’s very easy to end up writing code with security vulnerabilities, not to mention crashes due to mishandling pointers. The eBPF verifier helps out on the kernel side, but there is no equivalent protection for your user space code.

The good news is that there are libraries for other programming languages that interface to _libbpf_, or provide similar relocation functionality to allow for portable eBPF programs. Here are a few of the most popular ones.