# eBPF “Hello World” for a Network Interface

[[null|]]The examples in the previous chapter emitted the trace “Hello World” triggered by a system call kprobe; this time I’m going to show an eBPF program that writes a line of trace when triggered by the arrival of a network packet.

[[null|]]Packet processing is a very common application of eBPF. I’ll cover this in a lot more detail in [[ch08.xhtml#ebpf_for_networking|Chapter 8]], but for now it might be helpful to be aware of the basic idea of an eBPF program that is triggered for every packet of data that arrives on a network interface. The program can inspect and even modify the contents of that packet, and it makes a decision (or _verdict_) on what the kernel should do with that packet. The verdict could tell the kernel to carry on processing it as usual, drop it, or redirect it elsewhere.

In the simple example I’m showing here, the program doesn’t do anything with the network packet; it simply writes out the words _Hello World_ and a counter to the trace pipe every time a network packet is received.

The example program is in _chapter3/hello.bpf.c_. It’s a fairly common convention to put eBPF programs into filenames ending with _bpf.c_ to distinguish them from user space C code that might live in the same source code directory. Here’s the entire program:

```c
#include <linux/bpf.h>                           
#include <bpf/bpf_helpers.h>

int counter = 0;                                 

SEC("xdp")                                       
int hello(void *ctx) {                           
    bpf_printk("Hello World %d", counter);
    counter++;
    return XDP_PASS;
}

char LICENSE[] SEC("license") = "Dual BSD/GPL";  
```
1. [[null|]]This example starts by including some header files. Just in case you’re not familiar with C coding, every program has to include the header files that define any structures or functions the program is going to use. You can guess from the names that these header files are related to BPF.
2. This example shows how eBPF programs can use global variables. This counter will get incremented every time the program runs.
3. The macro `SEC()` defines a section called `xdp` that you’ll be able to see in the compiled object file. I’ll come back to how the section name is used in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], but for now you can simply think of it as defining that it’s an eXpress Data Path (XDP) type of eBPF program.
4. Here you can see the actual eBPF program. In eBPF, the program name is the function name, so this program is called `hello`. It uses a helper function, `bpf_printk`, to write a string of text, increments the global variable `counter`, and then returns the value `XDP_PASS`. This is the verdict indicating to the kernel that it should process this network packet as normal.
5. Finally there is another `SEC()` macro that defines a license string, and this is a crucial requirement for eBPF programs. Some of the BPF helper functions in the kernel are defined as “GPL only.” If you want to use any of these functions, your BPF code has to be declared as having a GPL-compatible license. The verifier (which we will discuss in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]]) will object if the declared license is not compatible with the functions a program uses. Certain eBPF program types, including those that use BPF LSM (which you’ll learn about in [[ch09.xhtml#ebpf_for_security|Chapter 9]]), are also [required to be GPL compatible](https://oreil.ly/ItahV).

> [!Note]
[[null|]]You might be wondering why the previous chapter used `bpf_trace_printk()` and this version uses `bpf_printk()`. The short answer is that BCC’s version is called `bpf_trace_printk()` and _libbpf_’s version is `bpf_printk()`, but both of those are wrappers around the kernel function `bpf_trace_printk()`. Andrii Nakryiko wrote a [good post](https://oreil.ly/9mNSY) on this on his blog.

This is an example of an eBPF program that attaches to the XDP hook point on a network interface. You can think of the XDP event being triggered the moment a network packet arrives inbound on a (physical or virtual) network interface.

>[!Note]
[[null|]]Some network cards support offloading XDP programs so that they can be executed on the network card itself. This means each network packet that arrives can be processed on the card, before it gets anywhere near the machine’s CPU. XDP programs can inspect and even modify each network packet, so this is very useful for doing things like DDoS protection, firewalling, or load balancing in a highly performant way. You’ll learn more about this in [[ch08.xhtml#ebpf_for_networking|Chapter 8]].

You have seen the C source code, so the next step is to compile it into an object the kernel can understand.[[null|]]