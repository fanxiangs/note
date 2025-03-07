# XDP

[[null|]][[null|]]You briefly met XDP (eXpress Data Path) eBPF programs in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]. In that example I loaded the eBPF program and attached it to the `eth0` interface using the following commands:

bpftool prog load hello.bpf.o /sys/fs/bpf/hello
bpftool net attach xdp id 540 dev eth0

It’s worth noting that XDP programs attach to a specific interface (or virtual interface), and you may very well have different XDP programs attached to different interfaces. In [[ch08.xhtml#ebpf_for_networking|Chapter 8]] you’ll learn more about how XDP programs can be offloaded to network cards or executed by network drivers.

XDP programs are another example of programs that can be managed using Linux network utilities—in this case, the `link` subcommand of [iproute2’s ip](https://oreil.ly/8Isau). The roughly equivalent command for loading and attaching the program to `eth0` would be this:

$ ip link set dev eth0 xdp obj hello.bpf.o sec xdp

This command reads the eBPF program marked as section `xdp` from the `hello.bpf.o` object and attaches it to the `eth0` network interface. The `ip link show` command for this interface now includes some information about the XDP program that’s attached to it:

2: eth0:  mtu 1500 xdpgeneric qdisc fq\_codel
state UP mode DEFAULT group default qlen 1000
    link/ether 52:55:55:3a:1b:a2 brd ff:ff:ff:ff:ff:ff
    prog/xdp id 1255 tag 9d0e949f89f1a82c jited

Removing the XDP program with `ip link` can be done like this:

$ ip link set dev eth0 xdp off

You’ll see a lot more about XDP programs and their applications in the next chapter.