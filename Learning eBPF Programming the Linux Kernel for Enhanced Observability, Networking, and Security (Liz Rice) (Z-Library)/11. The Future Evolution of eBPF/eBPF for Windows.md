# eBPF for Windows

[[null|]][[null|]][[null|]]Work is well underway at Microsoft to support [eBPF for Windows](https://oreil.ly/ArwkR). As I write this in the closing months of 2022, there are already [functional demos](https://oreil.ly/H-0dv) that show Cilium Layer 4 load balancing and eBPF-based connection tracking running on Windows.

I’ve said before that eBPF programming is kernel programming, and at first glance it might seem unintuitive that a program written to run in the Linux kernel and that has access to Linux kernel data structures would in any way be able to operate in an entirely different operating system. But in practice, particularly when it comes to networking, all operating systems will have quite a lot in common. A network packet has the same structure whether it was created on a Windows or Linux machine, and the layers of the network stack have to be handled the same way.

You’ll also recall that eBPF programs consist of a set of bytecode instructions that are processed by a virtual machine (VM) implemented within the kernel. That VM can be implemented within Windows too!

[[#architectural_overview_of_ebpf_for_wind|Figure 11-1]] shows the eBPF for Windows architectural overview, taken from [the project’s GitHub repo](https://oreil.ly/Ii4j2). As you can see from this diagram, eBPF for Windows reuses some open source components from the existing eBPF ecosystem, such as _libbpf_, and the support in Clang for producing eBPF bytecode. The Linux kernel is licensed under GPL, and Windows is proprietary, so the Windows project can’t reuse any parts of the Linux kernel’s implementation of the verifier.[^3] Instead, it uses the [PREVAIL verifier](https://vbpf.github.io) and the [uBPF JIT compiler](https://oreil.ly/btrkJ) (both of which are permissively licensed so that they can be used by a broader range of projects and organizations).

![Architectural overview of eBPF for Windows, taken from https://oreil.ly/HxKsu](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_1101.png)

###### Figure 11-1. Architectural overview of eBPF for Windows, adapted from [https://oreil.ly/HxKsu](https://oreil.ly/HxKsu)

One interesting difference is that eBPF code is verified and JIT-compiled in a Windows Secure environment in user space rather than within the kernel (the uBPF interpreter shown in the kernel in [[#architectural_overview_of_ebpf_for_wind|Figure 11-1]] is used only in debug builds and not production environments).

It would be unrealistic to expect that every single eBPF program written to run on Linux will work on Windows. But this isn’t so different from the challenge of getting eBPF programs to run on different Linux kernel versions: even with CO-RE support, internal kernel data structures can be changed as well as added or removed between versions. It is the eBPF programmer’s job to handle these possibilities gracefully.[[null|]][[null|]][[null|]]

Speaking of changes to the Linux kernel, what changes can we expect to see in eBPF in the coming years?