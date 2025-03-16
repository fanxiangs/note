# Testing BPF Programs

[[null|]][[null|]][[null|]][[null|]]There’s a `bpf()` command, [BPF_PROG_RUN](https://oreil.ly/Y2xPC), that allows for running an eBPF program from user space for test purposes.

`BPF_PROG_RUN` (currently) works only with a subset of BPF program types that are mostly networking related.

You can also get information about eBPF program performance with some built-in statistics information. Run the following command to enable it:

$ sysctl -w kernel.bpf\_stats\_enabled=1

This will show additional information in `bpftool`’s output about programs, like this:

$ bpftool prog list 
...
2179: raw\_tracepoint  name raw\_tp\_exec  tag 7f6d182e48b7ed38  gpl
        **run\_time\_ns 316876 run\_cnt 4**
        loaded\_at 2023-01-09T11:07:31+0000  uid 0
        xlated 216B  jited 264B  memlock 4096B  map\_ids 780,777
        btf\_id 953
        pids hello(19173)

The additional statistics are shown in bold, and here they show that the program has run four times, taking about 300 microseconds in total.

###### Note

Learn more from Quentin Monnet’s FOSDEM 2020 talk titled [“Tools and mechanisms to debug BPF programs.”](https://oreil.ly/I5Jhd)