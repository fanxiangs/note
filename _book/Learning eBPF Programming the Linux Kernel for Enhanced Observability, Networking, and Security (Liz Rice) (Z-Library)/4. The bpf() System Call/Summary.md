# Summary

In this chapter you saw how user space code uses the `bpf()` syscall to load eBPF programs and maps. You saw programs and maps being created using the `BPF_PROG_LOAD` and `BPF_MAP_CREATE` commands.

You learned that the kernel keeps track of the number of references to eBPF programs and maps, releasing them when the reference count drops to zero. You were also introduced to the concepts of pinning BPF objects to a filesystem and using BPF links to create additional references.

You saw an example of `BPF_MAP_UPDATE_ELEM` being used to create entries in a map from user space. There are similar commands—`BPF_MAP_LOOKUP_ELEM` and `BPF_MAP_DELETE_ELEM`—for retrieving and deleting values from a map. There is also the command `BPF_MAP_GET_NEXT_KEY` for finding the next key that’s present in a map. You can use this to iterate through all the valid entries.

You saw examples of user space programs making use of `perf_event_open()` and `ioctl()` for attaching eBPF programs to kprobe events. The attachment method can be very different for other types of eBPF programs, and some of them even use the `bpf()` system call. [[null|]]For example, there’s a `bpf(BPF_PROG_ATTACH)` syscall that can be used to attach cgroup programs, [[null|]]and `bpf(BPF_RAW_TRACEPOINT_OPEN)` for raw tracepoints (see Exercise 5 at the end of this chapter).

I also showed how you can use `BPF_MAP_GET_NEXT_ID`, `BPF_MAP_GET_FD_BY_ID`, and `BPF_OBJ_GET_INFO_BY_FD` to locate map (and other) objects held by the kernel.

There are some other `bpf()` commands that I haven’t covered in this chapter, but what you have seen here is enough to get a good overview.

You also saw some BTF data being loaded into the kernel, and I mentioned that `bpftool` uses this information to understand the format of data structures so that it can print them out nicely. I didn’t explain yet what BTF data looks like or how it’s used to make eBPF programs portable across kernel versions. That’s coming up in the next chapter.[[null|]]