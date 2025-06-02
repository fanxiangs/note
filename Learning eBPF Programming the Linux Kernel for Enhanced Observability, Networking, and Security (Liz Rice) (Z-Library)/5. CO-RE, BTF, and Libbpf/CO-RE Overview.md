# CO-RE Overview

[[null|]]The CO-RE approach consists of a few elements:[^2],[^3]

BTF

[BTF](https://oreil.ly/iRCuI) is a format for expressing the layout of data structures and function signatures. In CO-RE it’s used to determine any differences between the structures used at compilation time and at runtime. BTF is also used by tools like `bpftool` to dump data structures in human-readable formats. Linux kernels from 5.4 onward support BTF.

Kernel headers

The Linux kernel source code includes header files that describe the data structures it uses, and these headers can change between versions of Linux. eBPF programmers can choose to include individual header files, or, as you’ll see in this chapter, you can use `bpftool` to generate a header file called _vmlinux.h_ from a running system, containing all the data structure information about a kernel that a BPF program might need.

Compiler support

The [Clang compiler was enhanced](https://oreil.ly/6xFJm) so that when it compiles eBPF programs with the `-g` flag, it includes what are known as [[null|]]_CO-RE relocations_, derived from the BTF information describing the kernel data structures. The GCC compiler also added CO-RE support for BPF targets in [version 12](https://oreil.ly/_6PEE).

Library support for data structure relocations

At the point where a user space program loads an eBPF program into the kernel, the CO-RE approach requires the bytecode to be adjusted to compensate for any differences between the data structures present when it was compiled, and what’s on the destination machine where it’s about to run, based on the CO-RE relocation information compiled into the object. There are a few libraries that will take care of this: [libbpf](https://oreil.ly/E742u) was the original C library that includes this relocation capability, the Cilium eBPF library provides the same capability for Go programmers, and Aya does it for Rust.

Optionally, a BPF skeleton

[[null|]]A skeleton can be auto-generated from a compiled BPF object file, containing handy functions that user space code can call to manage the lifecycle of BPF programs—loading them into the kernel, attaching them to events, and so on. [[null|]]If you’re writing the user space code in C, you can generate the skeleton with `bpftool gen skeleton`. These functions are higher-level abstractions that can be more convenient for the developer than using the underlying library (_libbpf_, _cilium/ebpf_, etc.) directly.

###### Note

Andrii Nakryiko wrote an [excellent blog post](https://oreil.ly/aeQJo) that describes the background of CO-RE, as well as laying out how it works and how to use it. He also wrote the canonical [BPF CO-RE Reference Guide](https://oreil.ly/lbW_T), so please do read that if you’re embarking on writing code yourself. His [libbpf-bootstrap guide](https://oreil.ly/_jet-) to building an eBPF app from scratch with CO-RE + _libbpf_ + skeletons is another must-read.

Now that you have an overview of the elements of CO-RE, let’s dig in to see how they work, starting with an exploration of BTF.