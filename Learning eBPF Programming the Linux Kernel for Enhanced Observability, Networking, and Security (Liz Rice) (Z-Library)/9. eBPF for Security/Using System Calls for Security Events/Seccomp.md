# Seccomp

[[null|]][[null|]][[null|]]The name _seccomp_ is a contraction of “SECure COMPuting.” In its original, or “strict,” form, seccomp is used to limit the set of syscalls a process can use to a very small subset: `read()`, `write()`, `_exit()`, and `sigreturn()`. The intention of this strict mode was to allow users to run untrusted code (perhaps a program downloaded from the internet) without any possibility of that code doing malicious things.

Strict mode is very restrictive, and many applications need to use a much larger set of syscalls—but that doesn’t mean they need all 400 and more of them. It makes sense to allow a more flexible method for restricting the set that any given application can use. This is the reasoning behind the flavor of seccomp that most of us from the container world have encountered, which is more properly known as seccomp-bpf. Instead of having a fixed subset of syscalls that it permits, this mode of seccomp uses BPF code to filter the syscalls that are and aren’t allowed.

In seccomp-bpf, a set of BPF instructions are loaded that act as a filter. Each time a syscall is called, the filter is triggered. The filter code has access to the arguments that are passed to the syscall so that it can make decisions based on both the syscall itself and the arguments that have been passed to it. The outcome is one of a set of possible actions that include:

*   Allowing the syscall to go ahead
    
*   Returning an error code to the user space application
    
*   Killing the thread
    
*   Notifying a user space application (seccomp-unotify_)_ (as of kernel version 5.0)
    

##### Note

If you want to explore writing your own BPF filter code, Michael Kerrisk has some good examples at [https://oreil.ly/cJ6HL](https://oreil.ly/cJ6HL).

Some of the arguments passed to syscalls are pointers, and the BPF code in seccomp-bpf is not able to dereference these pointers. This limits the flexibility of a seccomp profile, as it can only use value arguments in its decision-making process. Also, it has to be applied to the process when it starts—you can’t modify the profile that is being applied to a given application process.

You may well have used seccomp-bpf without writing BPF code, as the code is often derived from a human-readable seccomp profile. [Docker’s default profile](https://oreil.ly/IT_Bf) is a good example. This is a general-purpose profile intended to be usable with pretty much any normal, containerized application. That inevitably means it allows most syscalls and disallows only a few that are unlikely to be appropriate in any application, `reboot()` being a great example.

[According to Aqua Security](https://oreil.ly/1xWmn), most containerized apps use somewhere in the range of 40 to 70 syscalls. For better security, it would be preferable to use a more constrained profile that is targeted at each specific application and only allows the syscalls it actually uses.