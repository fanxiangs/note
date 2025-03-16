# User Space Attachments

[[null|]][[null|]][[null|]]So far I have shown examples of eBPF programs attaching to events defined within the kernel’s source code. [[null|]]There are similar attachment points within user space code: uprobes and uretprobes for attaching to the entry and exit of user space functions, and user statically defined tracepoints (USDTs) for attaching to specified tracepoints within application code or user space libraries. These all use the `BPF_PROG_TYPE_KPROBE` program type.

##### Note

There are lots of public examples of programs attached to user space events. Here are a few from the BCC project:

*   The [bashreadline](https://oreil.ly/gDkaQ) and [funclatency tools](https://oreil.ly/zLT54) attach to u(ret)probe.
    
*   [USDT sample](https://oreil.ly/o894f) in BCC.
    

If you’re using _libbpf_, the `SEC()` macro lets you define the auto-attachment point for these user space probes. You’ll find the format required for the section name in the [libbpf documentation](https://oreil.ly/o0CBQ). For example, to attach a uprobe to the start of the `SSL_write()` function in OpenSSL, you would define the section for the eBPF program with the following:

    SEC

There are a few gotchas to be aware of when instrumenting user space code:

*   Notice that the path to this shared library in this example is architecture specific, so you may need corresponding architecture-specific definitions.
    
*   Unless you control the machine you’re running the code on, you can’t know what user space libraries and applications will be installed.
    
*   An application might be built as a standalone binary, so it won’t hit any probes you might attach within shared libraries.
    
*   Containers typically run with their own copy of a filesystem, with their own set of dependencies installed in it. The path to a shared library used by a container won’t be the same as the path to a shared library on the host machine.
    
*   Your eBPF program might need to be aware of the language in which an application was written. For example, in C the arguments to a function are generally passed using registers, but in Go they are passed using the stack,[^3] so the `pt_args` structure holding register information may be of less use.
    

That said, there are lots of useful tools that instrument user space applications with eBPF. For example, you can hook into the SSL library to trace out decrypted versions of encrypted information—we’ll explore this in more detail in the next chapter. Another example is continuous profiling of your applications, using tools such as [Parca](https://www.parca.dev).[[null|]][[null|]][[null|]]