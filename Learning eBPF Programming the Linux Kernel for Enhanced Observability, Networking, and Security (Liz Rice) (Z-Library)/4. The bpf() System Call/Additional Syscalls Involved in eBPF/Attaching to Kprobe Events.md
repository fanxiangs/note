# Attaching to Kprobe Events

[[null|]][[null|]][[null|]][[null|]]You’ve seen that file descriptor 6 was assigned to represent the eBPF program _hello_ once it was loaded into the kernel. To attach the eBPF program to an event, you also need a file descriptor representing that particular event. The following line from the `strace` output shows the creation of the file descriptor for the `execve()` kprobe:

perf\_event\_open({type=0x6 /\* PERF\_TYPE\_??? \*/, ...},...) = 7

According to the [manpage for the perf_event_open() syscall](https://oreil.ly/xpRJs)[[null|]], it “creates a file descriptor that allows measuring performance information.” You can see from the output that `strace` doesn’t know how to interpret the type parameter with the value `6`, but if you examine that manpage further, it describes how Linux supports dynamic types of [[null|]]Performance Measurement Unit:

> …there is a subdirectory per PMU instance under _/sys/bus/event\_source/devices_. In each subdirectory there is a type file whose content is an integer that can be used in the type field.

Sure enough, if you look under that directory, you’ll find a _kprobe/type_ file:

$ cat /sys/bus/event\_source/devices/kprobe/type
6

From this, you can see that the call to `perf_event_open()` has a type set to the value `6` to indicate that it’s a kprobe type of perf event.

Unfortunately, `strace` doesn’t output the details that would conclusively show that the kprobe is attached to the `execve()` syscall, but I hope there is enough evidence here to convince you that that’s what the file descriptor returned here represents.

The return code from `perf_event_open()` is `7`, and this represents the file descriptor for the kprobe’s perf event, and you know that file descriptor `6` represents the _hello_ eBPF program. [[null|]]The manpage for `perf_event_open()` also explains how to use `ioctl()` to create the attachment between the two:

> `PERF_EVENT_IOC_SET_BPF` \[...\] allows attaching a Berkeley Packet Filter (BPF) program to an existing kprobe tracepoint event. The argument is a BPF program file descriptor that was created by a previous `bpf(2)` system call.

This explains the following `ioctl()` syscall that you’ll see in the `strace` output, with arguments referring to the two file descriptors:

ioctl(7, PERF\_EVENT\_IOC\_SET\_BPF, 6)     = 0

There is also another `ioctl()` call that turns the kprobe event on:

ioctl(7, PERF\_EVENT\_IOC\_ENABLE, 0)      = 0

With this in place, the eBPF program should be triggered whenever `execve()` is run on this machine.