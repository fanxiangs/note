# Exercises

Here are some optional activities you might like to try (or think about) if you want to explore “Hello World” a bit further:

1.  Adapt the _hello-buffer.py_ eBPF program to output different trace messages for odd and even process IDs.
    
2.  Modify _hello-map.py_ so that the eBPF code gets triggered by more than one syscall. For example, `openat()` is commonly called to open files, and `write()` is called to write data to a file. You can start by attaching the _hello_ eBPF program to multiple syscall kprobes. Then try having modified versions of the _hello_ eBPF program for different syscalls, demonstrating that you can access the same map from multiple different programs.
    
3.  The _hello-tail.py_ eBPF program is an example of a program that attaches to the `sys_enter` raw tracepoint that is hit whenever _any_ syscall is called. Change _hello-map.py_ to show the total number of syscalls made by each user ID, by attaching it to that same `sys_enter` raw tracepoint.
    
    Here’s some example output I got after making that change:
    
    $ ./hello-map.py 
    ID 104: 6     ID 0: 225
    ID 104: 6     ID 101: 34    ID 100: 45    ID 0: 332     ID 501: 19
    ID 104: 6     ID 101: 34    ID 100: 45    ID 0: 368     ID 501: 38
    ID 104: 6     ID 101: 34    ID 100: 45    ID 0: 533     ID 501: 57
    
4.  The [RAW_TRACEPOINT_PROBE macro provided by BCC](https://oreil.ly/kh-j4) simplifies attaching to raw tracepoints, telling the user space BCC code to automatically attach it to a specified tracepoint. Try it in _hello-tail.py_, like this:
    
    *   Replace the definition of the `hello()` function with `RAW_TRACEPOINT_PROBE(sys_enter)`.
        
    *   Remove the explicit attachment call `b.attach_raw_tracepoint()` from the Python code.
        
    
    You should see that BCC automatically creates the attachment and the program works exactly the same. This is an example of the many convenient macros that BCC provides.
    
5.  You could further adapt _hello\_map.py_ so that the key in the hash table identifies a particular syscall (rather than a particular user). The output will show how many times that syscall has been called across the whole system.
    

[^1] I originally wrote this for a talk titled “The Beginner’s Guide to eBPF Programming.” You can find the original code along with links to the slides and video at [https://github.com/lizrice/ebpf-beginners](https://github.com/lizrice/ebpf-beginners).

[^2] There is a more performant way to attach eBPF programs to functions, available from kernel version 5.5 onward, that uses fentry (and the corresponding fexit instead of kretprobe for the exit from a function). I’ll discuss this later in the book, but for now I’m using kprobe to keep the example in this chapter as simple as possible.

[^3] I quite often use VScode remote to connect to a virtual machine in the cloud. This runs lots of node scripts on the virtual machine, which generates lots of tracing from this “Hello World” app.

[^4] Some commands (`echo` is a common example) might be shell built-ins that run as part of the shell process, rather than executing a new program. These won’t trigger the `execve()` event, so no trace will be generated.

[^5] C++ does, but not C.

[^6] The lower 32 bits are the _thread group ID_. For a single-threaded process, this is the same as the process ID, but additional threads for the process would be given different IDs. The docs for the GNU C library have a good description of the difference between [process and thread group IDs](https://oreil.ly/Wo9k3).

[^7] This is just example code, so I’m not worrying about cleaning up on keyboard interrupt or any other niceties!

[^8] This principle is often called “DRY” (“Don’t Repeat Yourself”), as popularized by [The Pragmatic Programmer](https://oreil.ly/QFich).

[^9] There are some 300 syscalls in Linux, and since I’m not using any recently added syscalls for this example, this is good enough.

[^10] Making tail calls from a BPF subprogram requires support from the JIT compiler, which you’ll meet in the next chapter. In the kernel version I used to write the examples in this book, only the JIT compiler on x86 has this support, although [support has been added to ARM in kernel 6.0](https://oreil.ly/KYUYS).