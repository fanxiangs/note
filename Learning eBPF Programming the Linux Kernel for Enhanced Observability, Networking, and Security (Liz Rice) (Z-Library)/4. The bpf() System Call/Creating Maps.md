# Creating Maps

[[null|]][[null|]]The next `bpf()` creates the `output` perf buffer map:

bpf(BPF\_MAP\_CREATE, {map\_type=BPF\_MAP\_TYPE\_PERF\_EVENT\_ARRAY, , key\_size=4, 
value\_size=4, max\_entries=4, ... map\_name="output", ...}, 128) = 4

[[null|]]You can probably guess from the command name `BPF_MAP_CREATE` that this call creates an eBPF map. You can see that the type of this map is `PERF_EVENT_ARRAY` and it is called `output`. The keys and values in this perf event map are 4 bytes long. There’s also a limit of four key–value pairs that can be held in this map, defined by the field `max_entries`; I’ll explain why there are four entries in this map later in this chapter. The return value of `4` is the file descriptor for the user space code to access the `output` map.

The next `bpf()` system call in the output creates the `config` map:

bpf(BPF\_MAP\_CREATE, {map\_type=BPF\_MAP\_TYPE\_HASH, key\_size=4, value\_size=12,
max\_entries=10240... map\_name="config", ...btf\_fd=3,...}, 128) = 5

This map is defined to be a hash table map, with keys that are 4 bytes long (which corresponds to a 32-bit integer that can be used to hold a user ID) and values that are 12 bytes long (which matches the length of the `msg_t` structure). I didn’t specify the size of the table, so it has been given BCC’s default size of 10,240 entries.

This `bpf()` system call also returns a file descriptor, `5`, which will be used to refer to this `config` map in future syscalls.

You can also see the field `btf_fd=3`, which tells the kernel to use the BTF file descriptor `3` that was obtained earlier. As you’ll see in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], BTF information describes the layout of data structures, and including this in the definition of the map means there’s information about the layout of the key and value types used in this map. This is used by tools like `bpftool` to pretty-print map dumps, making them human readable—you saw an example of this in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]].