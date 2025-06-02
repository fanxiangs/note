# Syscall-Tracking Security Tools

[[null|]]The [[null|]]best-known tool that falls into this category of syscall-tracking security tools is the CNCF project [Falco](https://falco.org), which provides security alerts. By default, Falco is installed as a kernel module, but there is also an eBPF version. Users can define [rules](https://oreil.ly/enufu) to determine what events are security relevant, and Falco can generate alerts in a variety of formats when events happen that don’t match the policies defined in these rules.

Both the kernel module driver and the eBPF-based driver attach to system calls. If you examine the [Falco eBPF programs on GitHub](https://oreil.ly/Q_cBD) you’ll see lines like the following that attach probes to raw syscall entry and exit points (plus a few other events, such as page faults):

    BPF_PROBE

Since eBPF programs can be loaded dynamically and can detect events triggered by preexisting processes, tools like Falco can apply policies to application workloads that are already running. Users can modify the set of rules being applied without having to modify the applications or their configuration. This is in contrast to seccomp profiles, which have to be applied to the application process when it is launched.

[[null|]]Unfortunately there is a problem with this approach of using syscall entry points for security tooling: there is a Time Of Check to Time Of Use (TOCTOU) issue.

When an eBPF program is triggered at the entry point to a system call, it can access the arguments that user space has passed to that system call. If those arguments are pointers, the kernel will need to copy the pointed-to data into its own data structures before acting on that data. As illustrated in [[#an_attacker_can_change_syscall_argument|Figure 9-2]], there is a window of opportunity for an attacker to modify this data, after it has been inspected by the eBPF program but before the kernel copies it. Thus, the data being acted on might not be the same as what was captured by the eBPF program.[^3]

![An attacker can change syscall arguments before they are accessed by the kernel](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0902.png)

##### Figure 9-2. An attacker can change syscall arguments before they are accessed by the kernel

The same window would apply for seccomp-bpf, were it not for the fact that in seccomp-bpf the program is not allowed to dereference the user space pointer, so it’s not possible to examine the data at all.

The TOCTOU issue does apply for seccomp\_unotify, a recently added mode of seccomp where a violation can be reported to user space. The [manpage for seccomp_unotify](https://oreil.ly/cwpki) explicitly notes that “It should thus be absolutely clear that the seccomp user-space notification mechanism _cannot_ be used to implement a security policy!”

The syscall entry point might be very convenient for observability purposes, but for a serious security tool it’s really not sufficient.

The [Sysmon for Linux tool](https://oreil.ly/pbtF3) addresses the TOCTOU window by attaching to both the entry and exit points for syscalls. Once the call has completed, it looks at the kernel’s data structures to get an accurate view. For example, if the syscall returns a file descriptor, the eBPF program attached to the exit can retrieve correct information about the object that the file descriptor represents by looking into the related process’s file descriptor table. While this approach can result in an accurate record of security-related activity, it can’t prevent an action from taking place, since the syscall has already completed by the time a check is made.[[null|]]

To be certain that it is inspecting the same information the kernel will act on, the eBPF program should be attached to an event that occurs after the parameters have been copied into kernel memory. Unfortunately, there is no single common place in the kernel to do this, as the data is handled differently in syscall-specific code. However, there is a well-defined interface where eBPF programs can be safely attached: the Linux Security Module (LSM) API.[[null|]] This requires a relatively new eBPF feature: BPF LSM.[[null|]][[null|]]