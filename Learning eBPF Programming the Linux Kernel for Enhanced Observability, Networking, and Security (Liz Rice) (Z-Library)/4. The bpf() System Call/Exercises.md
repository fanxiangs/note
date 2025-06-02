# Exercises

Here are a few things you can try if you’d like to explore the `bpf()` syscall further:

1.  [[null|]]Confirm that the `insn_cnt` field from a `BPF_PROG_LOAD` system call corresponds to the number of instructions that are output if you dump the translated eBPF bytecode for that program using `bpftool`. (This is as documented on the [manpage for the bpf() system call](https://oreil.ly/NJdIM).)
    
2.  Run two instances of the example program so that there are two maps called `config`. If you run `bpftool map dump name config`, the output will include information about the two different maps as well as their contents. Run this under `strace`, and follow the use of different file descriptors through the syscall output. Can you see where it’s retrieving information about a map and where it’s retrieving the key–value pairs stored within it?
    
3.  Use `bpftool map update` to modify the `config` map while one of the example programs is running. Use `sudo -u username` to check that these configuration changes are picked up by the eBPF program.
    
4.  While _hello-buffer-config.py_ is running, use `bpftool` to pin the program to the BPF filesystem, like this:
    
    bpftool prog pin name hello /sys/fs/bpf/hi
    
    Quit the running program, and check that the _hello_ program is still loaded in the kernel using `bpftool prog list`. You can clean up the link by removing the pin with `rm /sys/fs/bpf/hi`.
    
5.  Attaching to a raw tracepoint is considerably more straightforward at the syscall level than attaching to a kprobe, as it simply involves a `bpf()` syscall. Try converting _hello-buffer-config.py_ to attach to the raw tracepoint for `sys_enter`, using BCC’s `RAW_TRACEPOINT_PROBE` macro (if you did the exercises in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]], you’ll already have a suitable program you can use). You won’t need to explicitly attach the program in the Python code, as BCC will take care of it for you. Running this under `strace`, you should see a syscall similar to this:
    
    bpf(BPF\_RAW\_TRACEPOINT\_OPEN, {raw\_tracepoint={name="sys\_enter",
    prog\_fd=6}}, 128) = 7
    
    The tracepoint in the kernel has the name `sys_enter`, and the eBPF program with file descriptor `6` is being attached to it. From now on, whenever execution in the kernel reaches that tracepoint, it will trigger the eBPF program.
    
6.  Run the opensnoop application from [BCC’s set of libbpf tools](https://oreil.ly/D31R4). This tool sets up some BPF links that you can see with `bpftool`, like this:
    
    $ bpftool link list
    116: perf\_event  prog 1849  
            bpf\_cookie 0
            pids opensnoop(17711)
    117: perf\_event  prog 1851  
            bpf\_cookie 0
            pids opensnoop(17711)
    
    Confirm that the program IDs (1849 and 1851 in my example output here) match the output from listing the loaded eBPF programs:
    
    $ bpftool prog list
    ...
    1849: tracepoint  name tracepoint\_\_syscalls\_\_sys\_enter\_openat
            tag 8ee3432dcd98ffc3  gpl run\_time\_ns 95875 run\_cnt 121
            loaded\_at 2023-01-08T15:49:54+0000  uid 0
            xlated 240B  jited 264B  memlock 4096B  map\_ids 571,568
            btf\_id 710
            pids opensnoop(17711)
    1851: tracepoint  name tracepoint\_\_syscalls\_\_sys\_exit\_openat
            tag 387291c2fb839ac6  gpl run\_time\_ns 8515669 run\_cnt 120
            loaded\_at 2023-01-08T15:49:54+0000  uid 0
            xlated 696B  jited 744B  memlock 4096B  map\_ids 568,571,569
            btf\_id 710
            pids opensnoop(17711)
    
7.  While opensnoop is running, try pinning one of these links with `bpftool link pin id 116 /sys/fs/bpf/mylink` (using one of the link IDs you see output from `bpftool link list`). You should see that even after you terminate opensnoop, both the link and the corresponding program remain loaded in the kernel.
    
8.  If you skip ahead to the example code for [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], you’ll find a version of _hello-buffer-config.py_ written using the _libbpf_ library. This library automatically sets up a BPF link to the program that it loads into the kernel. Use `strace` to inspect the `bpf()` system calls that it makes, and see `bpf(BPF_LINK_CREATE)` system calls.
    

[^1] If you want to see the full set of BPF commands, they’re documented in the [linux/bpf.h](https://oreil.ly/Pyy7U) header file.

[^2] BTF was introduced upstream in the 5.1 kernel, but it has been back-ported on some Linux distributions, as you can see from [this discussion](https://oreil.ly/LjcPN).

[^3] These are defined in the `bpf_attach_type` enumerator in [linux/bpf.h](https://oreil.ly/AO1rc)_._

[^4] A reminder that for more information on the difference, read Andrii Nakryiko’s [“BPF ring buffer”](https://oreil.ly/XkpUF) blog post.