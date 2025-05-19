# Reading Map Elements

[[null|]][[null|]]At this point `bpftool` has a file descriptor reference to the map(s) it’s going to read from. Let’s look at the syscall sequence for reading that information:

bpf(BPF\_MAP\_GET\_NEXT\_KEY, {map\_fd=3, key=NULL,                    [[#list_id_4_11|]]
next\_key=0xaaaaf7a63960}, 24) = 0
bpf(BPF\_MAP\_LOOKUP\_ELEM, {map\_fd=3, key=0xaaaaf7a63960,           [[#list_id_4_12|]]
value=0xaaaaf7a63980, flags=BPF\_ANY}, 32) = 0
\[{                                                                [[#list_id_4_13|]]
        "key": 0,
        "value": {
            "message": "Hey root!"
        }
bpf(BPF\_MAP\_GET\_NEXT\_KEY, {map\_fd=3, key=0xaaaaf7a63960,          [[#list_id_4_14|]]
next\_key=0xaaaaf7a63960}, 24) = 0
bpf(BPF\_MAP\_LOOKUP\_ELEM, {map\_fd=3, key=0xaaaaf7a63960, 
value=0xaaaaf7a63980, flags=BPF\_ANY}, 32) = 0
    },{                                                   
        "key": 501,
        "value": {
            "message": "Hi user 501!"
        }
bpf(BPF\_MAP\_GET\_NEXT\_KEY, {map\_fd=3, key=0xaaaaf7a63960,          [[#list_id_4_15|]]
next\_key=0xaaaaf7a63960}, 24) = -1 ENOENT (No such file or directory)
    }                                                             [[#list_id_4_16|]]
\]
+++ exited with 0 +++

[[#code_id_4_11|]]

First, the application needs to find a valid key that is present in the map. It does this with the `BPF_MAP_GET_NEXT_KEY` flavor of the `bpf()` syscall. The `key` argument is a pointer to a key, and the syscall will return the next valid key _after_ this one. By passing in a NULL pointer, the application is requesting the first valid key in the map. The kernel writes the key into the location specified by the `next_key` pointer.

[[#code_id_4_12|]]

Given a key, the application requests the associated value, which gets written to the memory location specified by `value`.

[[#code_id_4_13|]]

At this point, `bpftool` has the contents of the first key–value pair, and it writes this information to the screen.

[[#code_id_4_14|]]

Here, `bpftool` moves on to the next key in the map, retrieves its value, and writes out this key–value pair to the screen.

[[#code_id_4_15|]]

The next call to `BPF_MAP_GET_NEXT_KEY` returns `ENOENT` to indicate that there are no more entries in the map.

[[#code_id_4_16|]]

Here, `bpftool` finalizes the output written to screen and exits.

Notice that here, `bpftool` has been assigned file descriptor `3` to correspond to the `config` map. This is the same map that _hello-buffer-config.py_ refers to with file descriptor `4`. As I’ve mentioned already, file descriptors are process specific.

This analysis of how `bpftool` behaves shows how a user space program can iterate through the available maps and through the key–value pairs stored in a map.[[null|]][[null|]]