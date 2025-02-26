# Modifying a Map from User Space

[[null|]][[null|]]You already saw the line in the Python user space source code that configures special messages that will be displayed for the root user with user ID 0, and for the user with ID 501:

    b

You can see these entries being defined in the map through syscalls like this:

bpf(BPF\_MAP\_UPDATE\_ELEM, {map\_fd=5, key=0xffffa7842490, value=0xffffa7a2b410,
flags=BPF\_ANY}, 128) = 0

[[null|]]The `BPF_MAP_UPDATE_ELEM` command updates the key–value pair in a map. The `BPF_ANY` flag indicates that if the key doesn’t already exist in this map, it should be created. There are two of these calls, corresponding to the two entries configured for two different user IDs.

The `map_fd` field identifies which map is being operated on. You can see that in this case it’s `5`, which is the file descriptor value returned earlier when the `config` map was created.

File descriptors are assigned by the kernel for a particular process, so this value of `5` is only valid for this particular user space process in which the Python program is running. However, multiple user space programs (and multiple eBPF programs in the kernel) can all access the same map. Two user space programs that access the same map structure in the kernel might very well be assigned different file descriptor values; equally, two user space programs might have the same file descriptor value for entirely different maps.

Both the key and the value are pointers, so you can’t tell the numeric value of either the key or the value from this `strace` output. [[null|]]You could, however, use `bpftool` to view the map’s contents and see something like this:

$ bpftool map dump name config
\[{
        "key": 0,
        "value": {
            "message": "Hey root!"
        }
    },{
        "key": 501,
        "value": {
            "message": "Hi user 501!"
        }
    }
\]

How does `bpftool` know how to format this output? For example, how does it know the value is a structure, with a field called `message` that contains a string? The answer is that it uses the definitions in the BTF information included on the `BPF_MAP_CREATE` syscall that defined this map. You’ll see more details on how BTF conveys this information in the next chapter.

You’ve now seen how user space interacts with the kernel to load programs and maps and to update the information in a map. In the sequence of syscalls you have seen up to this point, the program hasn’t yet been attached to an event. This step has to happen; otherwise, the program will never be triggered.

Fair warning: different types of eBPF programs get attached to different events in a variety of different ways! Later in this chapter I’ll show you the syscalls used in this example to attach to the kprobe event, and in this case it doesn’t involve `bpf()`. In contrast, in the exercises at the end of this chapter I will show you another example where a `bpf()` syscall is used to attach a program to a raw tracepoint event.

Before we get to those details, I’d like to discuss what happens when you quit running the program. You’ll find that the program and maps are automatically unloaded, and this happens because the kernel is keeping track of them using _reference counts_.[[null|]][[null|]]