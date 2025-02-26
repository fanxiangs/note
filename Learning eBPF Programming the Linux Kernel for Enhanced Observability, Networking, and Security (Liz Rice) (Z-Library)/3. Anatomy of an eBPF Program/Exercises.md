# Exercises

Here are a few things to try if you want to explore BPF programs further:

1.  Try using `ip link` commands like the following to attach and detach the XDP program:
    
    $ ip link set dev eth0 xdp obj hello.bpf.o sec xdp
    $ ip link set dev eth0 xdp off
    
2.  Run any of the BCC examples from [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]]. While the program is running, use a second terminal window to inspect the loaded program using `bpftool`. Here’s an example of what I saw by running the _hello-map.py_ example:
    
    $ bpftool prog show name hello 
    197: kprobe  name hello  tag ba73a317e9480a37  gpl
            loaded\_at 2022-08-22T08:46:22+0000  uid 0
            xlated 296B  jited 328B  memlock 4096B  map\_ids 65
            btf\_id 179
            pids hello-map.py(2785)
    
    You can also use `bpftool prog dump` commands to see the bytecode and machine code versions of those programs.
    
3.  Run _hello-tail.py_ from the _chapter2_ directory, and while it’s running, take a look at the programs it loaded. You’ll see that each tail call program is listed individually, like this:
    
    $ bpftool prog list 
    ...
    120: raw\_tracepoint  name hello  tag b6bfd0e76e7f9aac  gpl
            loaded\_at 2023-01-05T14:35:32+0000  uid 0
            xlated 160B  jited 272B  memlock 4096B  map\_ids 29
            btf\_id 124
            pids hello-tail.py(3590)
    121: raw\_tracepoint  name ignore\_opcode  tag a04f5eef06a7f555  gpl
            loaded\_at 2023-01-05T14:35:32+0000  uid 0
            xlated 16B  jited 72B  memlock 4096B
            btf\_id 124
            pids hello-tail.py(3590)
    122: raw\_tracepoint  name hello\_exec  tag 931f578bd09da154  gpl
            loaded\_at 2023-01-05T14:35:32+0000  uid 0
            xlated 112B  jited 168B  memlock 4096B
            btf\_id 124
            pids hello-tail.py(3590)
    123: raw\_tracepoint  name hello\_timer  tag 6c3378ebb7d3a617  gpl
            loaded\_at 2023-01-05T14:35:32+0000  uid 0
            xlated 336B  jited 356B  memlock 4096B
            btf\_id 124
            pids hello-tail.py(3590)
    
    You could also use `bpftool prog dump xlated` to look at the bytecode instructions and compare them to what you saw in [[#bpf_to_bpf_calls|“BPF to BPF Calls”]].
    
4.  _Be careful with this one, as it may be best to simply think about why this happens rather than trying it!_ If you return a `0` value from an XDP program, this corresponds to `XDP_ABORTED`, which tells the kernel to abort any further processing of this packet. This might seem a bit counterintuitive given that the `0` value usually indicates success in C, but that’s how it is. So, if you try modifying the program to return `0` and attach it to a virtual machine’s `eth0` interface, all network packets will get dropped. This will be somewhat unfortunate if you’re using SSH to attach to that machine, and you’ll likely have to reboot the machine to regain access!
    
    You could run the program within a container so that the XDP program is attached to a virtual Ethernet interface that only affects that container and not the whole virtual machine. There’s an example of doing this at [https://github.com/lizrice/lb-from-scratch](https://github.com/lizrice/lb-from-scratch).
    

[^1] Increasingly, eBPF programs are also being written in Rust, since the Rust compiler supports eBPF bytecode as a target.

[^2] There are a few instructions where the operation is “modified” by the value of other fields in the instruction. For example, there are a set of [atomic instructions](https://oreil.ly/oyTI7) introduced in kernel 5.12 that include an arithmetic operation (`ADD`, `AND`, `OR`, `XOR`) that is specified in the `imm` field.

[^3] The `-g` flag is required to generate BTF information that you’ll need for CO-RE eBPF programs, which I’ll cover in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]].

[^4] In general, this is optional—eBPF programs can be loaded into the kernel without being pinned to a file location—but it’s not optional for `bpftool`, which always has to pin the programs it loads. The reason for this is covered further in [[ch04.xhtml#bpf_program_and_map_references|“BPF Program and Map References”]].

[^5] The kernel setting `CONFIG_BPF_JIT` needs to be enabled to take advantage of JIT compilation, and it can be enabled or disabled at runtime with the `net.core.bpf_jit_enable sysctl` setting. See [the docs](https://oreil.ly/4-xi6) for more information on JIT support on different chip architectures.

[^6] Here, _bss_ stands for “block started by symbol.”