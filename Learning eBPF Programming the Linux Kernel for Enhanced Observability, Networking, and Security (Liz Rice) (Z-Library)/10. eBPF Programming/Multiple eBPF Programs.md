# Multiple eBPF Programs

[[null|]]An eBPF program is a function attached to an event in the kernel. Many applications need to track more than one event to achieve their goals. [[null|]]A simple example of this is opensnoop.[^6] I covered the `bpftrace` version of this early in this chapter, and you saw that it attaches BPF programs to four different syscall tracepoints:

*   `syscall_enter_open`
    
*   `syscall_exit_open`
    
*   `syscall_enter_openat`
    
*   `syscall_exit_openat`
    

These are the entry and exit points to the kernel’s handling of the `open()` and `openat()` system calls. These two system calls can be used for opening files, and the opensnoop tool tracks both of them.

But why does it need to track both entry and exit for these system calls? The entry points are used because that’s when the system call arguments are available, and these include the filename and any flags being passed to the `open[at]` syscall. But at that stage it’s too soon to know whether the file will be opened successfully or not. That explains why it’s necessary to have eBPF programs attached to the exit points too.

If you look at the [libbpf-tools version of opensnoop](https://oreil.ly/IOty_), you’ll see there’s just one user space program, and it loads all four eBPF programs into the kernel and attaches them to their events. The eBPF programs themselves are essentially independent, but they use eBPF maps to coordinate among themselves.

A complex application might even need to add and remove eBPF programs dynamically throughout a long period of time. There may not even be a fixed number of eBPF programs for any given application. For example, Cilium attaches eBPF programs to each virtual networking interface, and in a Kubernetes environment these interfaces come and go depending on how many pods are running.

Most of the libraries in this chapter handle this multiplicity of eBPF programs automatically. For example, _libbpf_ and _ebpf-go_ generate skeleton code that will load _all_ the programs and maps from the bytecode in an object file or buffer in one function call. They also generate finer-granularity functions so that you can manipulate programs and maps individually.