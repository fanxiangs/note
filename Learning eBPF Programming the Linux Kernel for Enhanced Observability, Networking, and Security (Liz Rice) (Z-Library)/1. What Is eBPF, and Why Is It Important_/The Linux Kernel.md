# The Linux Kernel

[[null|]][[null|]]To understand eBPF you’ll need a solid grasp of the difference between the kernel and user space in Linux. I covered this in my report “What Is eBPF?”[^6] and I’ve adapted some of that content for the next few paragraphs.

[[null|]]The Linux kernel is the software layer between your applications and the hardware they’re running on. [[null|]]Applications run in an unprivileged layer called _user space_, which can’t access hardware directly. [[null|]]Instead, an application makes requests using the system call (syscall) interface to request the kernel to act on its behalf. That hardware access can involve reading and writing to files, sending or receiving network traffic, or even just accessing memory. The kernel is also responsible for coordinating concurrent processes, enabling many applications to run at once. This is illustrated in [[#applications_in_user_space_use_the_sysc|Figure 1-1]].

As application developers, we typically don’t use the system call interface directly, because programming languages give us high-level abstractions and standard libraries that are easier interfaces to program. As a result, a lot of people are blissfully unaware of how much the kernel is doing while our programs run. If you want to get a sense of how often the kernel is invoked, you can use the `strace` utility to show all the system calls an application makes.

![Applications in user space use the syscall interface to make requests to the kernel](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0101.png)

###### Figure 1-1. Applications in user space use the syscall interface to make requests to the kernel

Here’s an example, where using `cat` to echo the word _hello_ to the screen involves more than 100 system calls:

$ strace -c echo "hello"
hello
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 24.62    0.001693          56        30        12 openat
 17.49    0.001203          60        20           mmap
 15.92    0.001095          57        19           newfstatat
 15.66    0.001077          53        20           close
 10.35    0.000712         712         1           execve
  3.04    0.000209          52         4           mprotect
  2.52    0.000173          57         3           read
  2.33    0.000160          53         3           brk
  2.09    0.000144          48         3           munmap
  1.11    0.000076          76         1           write
  0.96    0.000066          66         1         1 faccessat
  0.76    0.000052          52         1           getrandom
  0.68    0.000047          47         1           rseq
  0.65    0.000045          45         1           set\_robust\_list
  0.63    0.000043          43         1           prlimit64
  0.61    0.000042          42         1           set\_tid\_address
  0.58    0.000040          40         1           futex
------ ----------- ----------- --------- --------- ----------------
100.00    0.006877          61       111        13 total

Because applications rely so heavily on the kernel, it means we can learn a lot about how an application behaves if we can observe its interactions with the kernel. With eBPF we can add instrumentation into the kernel to get these insights.

For example, if you are able to intercept the system call for opening files, you can see exactly which files any application accesses. But how could you do that interception? Let’s consider what would be involved if we wanted to modify the kernel, adding new code to create some kind of output whenever that system call is invoked.[[null|]][[null|]]