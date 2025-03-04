# BCC’s “Hello World”

The following is the full source code of _hello.py_, an eBPF “Hello World” application[^1] written using BCC’s Python library:
```python
#!/usr/bin/python  
from bcc import BPF

program = r"""
int hello(void *ctx) {
    bpf_trace_printk("Hello World!");
    return 0;
}
"""

b = BPF(text=program)
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

b.trace_print()

```

This code consists of two parts: the eBPF program itself that will run in the kernel, and some user space code that loads the eBPF program into the kernel and reads out the trace that it generates. As you can see in [[#the_user_space_and_kernel_components_of|Figure 2-1]], _hello.py_ is the user space part of this application, and `hello()` is the eBPF program that runs in the kernel.

![The user space and kernel components of “Hello World”](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0201.png)

###### Figure 2-1. The user space and kernel components of “Hello World”

Let’s dig into each line of the source code to understand it better.

The first line tells you this is Python code, and the program that can run it is the Python interpreter (_/usr/bin/python_).

The eBPF program itself is written in C code, and it’s this part:

```c
int hello(void *ctx) {
    bpf_trace_printk("Hello World!");
    return 0;
}
```

[[null|]]All the eBPF program does is use a helper function, `bpf_trace_printk()`, to write a message. Helper functions are another feature that distinguishes “extended” BPF from its “classic” predecessor. They are a set of functions that eBPF programs can call to interact with the system; I’ll discuss them further in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]]. For now you can just think of this as printing a line of text.

The entire eBPF program is defined as a string called `program` in the Python code. This C program needs to be compiled before it can be executed, but BCC takes care of that for you. (You’ll see how to compile eBPF programs yourself in the next chapter.) All you need to do is pass this string as a parameter when creating a BPF object, as in the following line:

```c
b = BPF(text=program)
```

==eBPF programs need to be attached to an event==, and for this example I’ve chosen to attach to the system call `execve`, which is the syscall used to execute a program. Whenever anything or anyone starts a new program executing on this machine, that will call `execve()`, which will trigger the eBPF program. Although the “execve()” name is a standard interface in Linux, the name of the function that implements it in the kernel depends on the chip architecture, but BCC gives us a convenient way to look up the function name for the machine we’re running on:
```c
syscall = b.get_syscall_fnname("execve")
```

Now, `syscall` represents the name of the kernel function I’m going to attach to, using a kprobe (you were introduced to the concept of kprobes in [[ch01.xhtml#what_is_ebpf_and_why_is_it_importantque|Chapter 1]]).[^2] You can attach the `hello` function to that event, like this:

```c
b.attach_kprobe(event=syscall, fn_name="hello")
```

At this point, the eBPF program is loaded into the kernel and attached to an event, so the program will be triggered whenever a new executable gets launched on the machine. All that’s left to do in the Python code is to read the tracing that is output by the kernel and write it on the screen:

b.trace\_print()

This `trace_print()` function will loop indefinitely (until you stop the program, perhaps with Ctrl+C), displaying any trace.

[[#quotation_markhello_worldquotation_mark|Figure 2-2]] illustrates this code. The Python program compiles the C code, loads it into the kernel, and attaches it to the `execve` syscall kprobe. Whenever any application on this (virtual) machine calls `execve()`, it triggers the eBPF `hello()` program, which writes a line of trace into a specific pseudofile. (I’ll cover where that pseudofile is later in this chapter.) The Python program reads the trace message from the pseudofile and displays it to the user.

![“Hello World” in operation](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0202.png)

###### Figure 2-2. “Hello World” in operation