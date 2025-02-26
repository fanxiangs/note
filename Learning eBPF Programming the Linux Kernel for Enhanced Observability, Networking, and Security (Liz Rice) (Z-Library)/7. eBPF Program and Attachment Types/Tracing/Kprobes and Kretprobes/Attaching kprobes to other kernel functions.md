# Attaching kprobes to other kernel functions

[[null|]]You can find lots of examples where eBPF-based tools use kprobes to attach to system calls, but, as mentioned earlier, kprobes can also be attached to any noninlined function in the kernel. I’ve provided an example in _hello.bpf.c_ that attaches a kprobe to the function `do_execve()`, and it’s defined like this:

    SEC

Because `do_execve()` isn’t a system call, there are a few differences between this and the previous example:

*   The format of the SEC name is identical to the previous version attached to the syscall entry point, but there is no need to define platform-specific variants because `do_execve()`, like most kernel functions, is common to all platforms.
    
*   I used the `BPF_KPROBE` macro rather than `BPF_KPROBE_SYSCALL`. The intent is exactly the same, just that the latter handles syscall parameters.
    
*   There is another important difference: the `pathname` parameter to the syscall is a pointer to a string `(char *)`, but for this function the parameter is called `filename`, and it’s a pointer to a `struct filename`, which is a data structure used within the kernel.
    

You might well be wondering how I knew to use this type for this parameter. I’ll show you. The `do_execve()` function in the kernel has the following signature:

    int

I chose to ignore the `do_execve()` parameters `__argv` and `__envp`, and only declare the `filename` argument, using the type `struct filename *` to match the kernel function’s definition. Given the way arguments are laid out sequentially in memory, it’s OK to ignore the last _n_ parameters, but you can’t ignore an earlier argument in the list if you want to use a later one.

This `filename` structure is defined internal to the kernel, and it’s an illustration of how eBPF programming is kernel programming: I had to look up the definition of `do_execve()` to find its arguments, and the definition of `struct filename`. The name of the executable that is about to be run is pointed to by `filename->name`. I’m retrieving this name in the example code with the following lines:

    const

So to recap: the context parameter to a syscall kprobe is a structure representing the values passed by user space into the syscall. The context parameter to a “regular” (nonsyscall) kprobe is a structure representing the parameters passed to the called function by whatever kernel code is calling it, so the structure depends on the function definition.

Kretprobes are very similar to kprobes, except that they are triggered when a function returns and can access the return value instead of the arguments.

Kprobes and kretprobes are a reasonable way to hook into kernel functions, but there’s a newer option you should consider if you’re running on recent kernels.[[null|]][[null|]]