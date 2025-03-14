# Dynamic Loading of eBPF Programs

eBPF programs can be loaded into and removed from the kernel dynamically. Once they are attached to an event, they’ll be triggered by that event regardless of what caused that event to occur. For example, if you attach a program to the syscall for opening files, it will be triggered whenever any process tries to open a file. It doesn’t matter whether that process was already running when the program was loaded. This is a huge advantage compared to upgrading the kernel and then having to reboot the machine to use its new functionality.

This leads to one of the great strengths of observability or security tooling that uses eBPF—it instantly gets visibility over everything that’s happening on the machine. In environments running containers, that includes visibility over all processes running inside those containers as well as on the host machine. I’ll dig into the consequences of this for cloud native deployments later in this chapter.

Additionally, as illustrated in [[#adding_kernel_features_with_ebpf_left_p|Figure 1-3]], people can create new kernel functionality very quickly through eBPF without requiring every other Linux user to accept the same changes.

![Adding kernel features with eBPF (cartoon by Vadim Shchekoldin, Isovalent)](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0103.png)

###### Figure 1-3. Adding kernel features with eBPF (cartoon by Vadim Shchekoldin, Isovalent)