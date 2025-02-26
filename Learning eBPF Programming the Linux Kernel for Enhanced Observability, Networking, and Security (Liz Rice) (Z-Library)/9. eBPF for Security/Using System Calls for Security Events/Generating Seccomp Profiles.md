# Generating Seccomp Profiles

[[null|]][[null|]][[null|]]If you ask the average app developer to tell you what syscalls one of their programs makes, you’re likely to get a blank look. That’s not intended to be insulting. It’s just that most developers write in programming languages that give them higher-level abstractions far removed from the details of syscalls. For example, they might know what files their application opens, but it’s less likely that they could tell you whether they are opened using `open()` or `openat()`_._ This makes it unlikely that you’ll get a positive response if you ask the developer to handcraft an appropriate seccomp profile along with their application code.

Automation is the way forward: the idea is to use a tool to record the set of syscalls an application makes. In the early days, seccomp profiles were generally compiled using `strace` to gather the set of syscalls an application calls.[^1] This isn’t a wonderful solution in the cloud native age, as there’s no easy way to point `strace` at a specific container or Kubernetes pod. It would also be more helpful to generate the profile not just as a list of syscalls, but in the JSON format that Kubernetes and OCI-compatible container runtimes can take as input. There are a couple of tools that do this, using eBPF to gather information about all the syscalls being called:

*   [Inspektor Gadget](https://www.inspektor-gadget.io) includes a seccomp profiler that allows you to generate a custom seccomp profile for the containers in a Kubernetes pod.[^2]
    
*   Red Hat created a seccomp profiler in the form of an [OCI runtime hook](https://oreil.ly/nC8vM).
    

With these profilers, you need to run the application for some arbitrary amount of time to generate a profile that includes the full list of the syscalls it might legitimately call. As discussed earlier in this chapter, this list needs to include error paths. If your application can’t behave correctly under error conditions because the syscalls it needs to call are blocked, this might cause a bigger problem. And since seccomp profiles deal with a lower abstraction level than most developers are familiar with, it’s hard to review them manually to see if they cover all the right cases.

Taking the OCI runtime hook as an example, an eBPF program is [attached to the syscall_enter raw tracepoint](https://oreil.ly/sbWSc) and maintains an eBPF map that keeps track of [which syscalls have been seen](https://oreil.ly/czUM7). The user space parts of this tool are written in Go and use the [iovisor/gobpf library](https://oreil.ly/sYCT3). (I’ll discuss this and other Golang libraries for eBPF in [[ch10.xhtml#ebpf_programming|Chapter 10]].)

The following are the [lines of code](https://oreil.ly/DOShA) from the OCI runtime hook that load the eBPF program into the kernel and attach it to the tracepoint (a few lines have been omitted for brevity):

    src

[[#code_id_9_1|]]

This line does something quite interesting: it replaces a variable named `$PARENT_PID` in the eBPF source code with a numeric process ID. This is a common pattern, and it indicates that this tool will load individual eBPF programs for each process being instrumented.

[[#code_id_9_2|]]

Here, an eBPF program called `enter_trace` gets loaded into the kernel.

[[#code_id_9_3|]]

The `enter_trace` program gets attached to the tracepoint `raw_syscalls:sys_enter`. This is the tracepoint at the point of entry to any syscall, which you’ve encountered in earlier examples. Whenever any user space code makes a syscall, this tracepoint will be hit.

These profilers use eBPF code attached to `sys_enter` to keep track of the set of syscalls that have been used, and they generate a seccomp profile to be used with seccomp, which does the actual job of enforcing the profile. The next class of eBPF tools we’ll consider also attach to `sys_enter`, but they use syscalls to track the behavior of an application and compare it against security policies.[[null|]][[null|]][[null|]]