# BPF Maps

[[null|]]A [[null|]]_map_ is a data structure that can be accessed from an eBPF program and from user space. Maps are one of the really significant features that distinguish extended BPF from its classic predecessor. (You might think this would mean they are commonly referred to as “eBPF maps,” but you’ll frequently see “BPF maps.” As is generally the case, both terms are used interchangeably.)

Maps can be used to share data among multiple eBPF programs or to communicate between a user space application and eBPF code running in the kernel. Typical uses include the following:

*   User space writing configuration information to be retrieved by an eBPF program
*   An eBPF program storing state, for later retrieval by another eBPF program (or a future run of the same program)
*   An eBPF program writing results or metrics into a map, for retrieval by the user space app that will present results
    

There are various types of BPF maps defined in Linux’s [uapi/linux/bpf.h file](https://oreil.ly/1s1GM), and there is some information about them in the [kernel docs](https://oreil.ly/5oUW7). In general they are all key–value stores, and in this chapter you’ll see examples of maps for hash tables, perf and ring buffers, and arrays of eBPF programs.

Some map types are defined as arrays, which always have a 4-byte index as the key type; other maps are hash tables that can use some arbitrary data type as the key.

There are map types that are optimized for particular types of operations, such as [first-in-first-out queues](https://oreil.ly/VSoEp), [first-in-last-out stacks](https://oreil.ly/VSoEp), [least-recently-used data storage](https://oreil.ly/vpsun), [longest-prefix matching](https://oreil.ly/hZ5aM), and [Bloom filters](https://oreil.ly/DzCTK) (a probabilistic data structure designed to provide very fast results on whether an element exists).

Some eBPF map types hold information about specific types of objects. For example, [sockmaps](https://oreil.ly/UUTHO) and [devmaps](https://oreil.ly/jzKYh) hold information about sockets and network devices and are used by network-related eBPF programs to redirect traffic. A program array map stores a set of indexed eBPF programs, and (as you’ll see later in this chapter) this is used to implement tail calls, where one program can call another. There’s even a [map-of-maps type](https://oreil.ly/038tN) to support storing information about maps.

Some map types have per-CPU variants, which is to say that the kernel uses a different block of memory for each CPU core’s version of that map. This might have you wondering about concurrency concerns for maps that are _not_ per-CPU, where multiple CPU cores could be accessing the same map simultaneously. Spin lock support for (some) maps was added in kernel version 5.1, and we’ll return to this subject in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]].

The next example (_chapter2/hello-map.py_ in the [GitHub repository](https://github.com/lizrice/learning-ebpf)) shows some basic operations using a hash table map. It also demonstrates some of BCC’s convenient abstractions that make it very easy to use maps.