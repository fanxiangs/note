# Attaching to an Event

[[null|]][[null|]][[null|]]The program type has to match the type of event it’s being attached to; you’ll learn more about this in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]]. [[null|]]In this case it’s an XDP program, and you can use `bpftool` to attach the example eBPF program to the XDP event on a network interface, like this:

$ bpftool net attach xdp id 540 dev eth0

###### Note

At the time of this writing, the `bpftool` utility doesn’t support the ability to attach all program types, but it has [recently been extended](https://oreil.ly/Tt99p) to auto-attach k(ret)probes, u(ret)probes, and tracepoints.

[[null|]]Here I have used the program’s ID of 540, but you can also use the name (provided it is unique) or tag to identify the program being attached. In this example, I have attached the program to the network interface `eth0`.

You can view all the network-attached eBPF programs using `bpftool`:

$ bpftool net list 
xdp:
eth0(2) driver id 540

tc:

flow\_dissector:

The program with ID 540 is attached to the XDP event on the `eth0` interface. This output also gives some clues about some other potential events in the network stack that you can attach eBPF programs to: `tc` and `flow_dissector`. More on this in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].

[[null|]]You can also inspect the network interfaces using `ip link`, and you’ll see output that looks something like this (some details have been removed for clarity):

1: lo:  mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
group default qlen 1000
    ...
2: eth0:  mtu 1500 xdp qdisc fq\_codel state UP
mode DEFAULT group default qlen 1000
    ...
    prog/xdp id 540 tag 9d0e949f89f1a82c jited 
    ...

In this example there are two interfaces: the loopback interface `lo`, which is used to send traffic to processes on this machine; and the `eth0` interface, which connects this machine to the outside world. This output also shows that `eth0` has a JIT-compiled eBPF program, with identity `540` and tag `9d0e949f89f1a82c`, attached to its XDP hook.

###### Note

You can also use `ip link` to attach and detach XDP programs to a network interface. I have included this as an exercise at the end of this chapter, and there are further examples in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].

At this point, the _hello_ eBPF program should be producing trace output every time a network packet is received. You can check this out by running `cat /sys/kernel/debug/tracing/trace_pipe`. This should show a lot of output that looks similar to this:

\-0       \[003\] d.s.. 655370.944105: bpf\_trace\_printk: Hello World 4531
\-0       \[003\] d.s.. 655370.944587: bpf\_trace\_printk: Hello World 4532
\-0       \[003\] d.s.. 655370.944896: bpf\_trace\_printk: Hello World 4533

[[null|]]If you’re struggling to remember the location of the trace pipe, you can get the same output using the command `bpftool prog tracelog`.

In comparison to the output you saw in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]], this time there is no command or process ID associated with each of these events; instead, you see `-0` at the start of each line of trace. In [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]], each syscall event happened because a process executing a command in user space made a call to the syscall API. That process ID and command are part of the context in which the eBPF program was executed. But in the example here, the XDP event happens due to the arrival of a network packet. There is no user space process associated with this packet—at the point the _hello_ eBPF program is triggered, the system hasn’t done anything with the packet other than receive it in memory, and it has no idea what the packet is or where it’s going.

You can see that the counter value that is traced out is being incremented by one each time, as expected. In the source code, `counter` is a global variable. Let’s see how that is implemented in eBPF using a map.[[null|]][[null|]][[null|]]