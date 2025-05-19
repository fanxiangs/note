# Summary

In this chapter you saw that various eBPF program types are used to attach into different hook points in the kernel. If you want to write code that responds to a particular event, you’ll need to determine the program type(s) that are appropriate for hooking onto that event. The context passed into the program depends on the program type, and the kernel may also respond differently to the return code from your program, depending on its type.

The example code for this chapter mostly focused on perf-related (tracing) events. In the next two chapters you’ll see more details on different eBPF program types used for networking and security applications.[[null|]][[null|]]