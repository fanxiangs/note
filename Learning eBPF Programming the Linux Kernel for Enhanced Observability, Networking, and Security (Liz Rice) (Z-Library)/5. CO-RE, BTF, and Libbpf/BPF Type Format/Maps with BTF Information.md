# Maps with BTF Information

[[null|]][[null|]]You’ve just seen the BTF information associated with a map. Now let’s see how this BTF data is passed to the kernel when the map is created.

You saw in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]] that maps are created using the `bpf(BPF_MAP_CREATE)` syscall. This takes a `bpf_attr` structure as a parameter, [defined in the kernel](https://oreil.ly/PLrYG) like this (some details omitted):

    struct

Before the introduction of BTF, the `btf_*` fields weren’t present in this `bpf_attr` structure, and the kernel had no knowledge of the structure of keys or values. The `key_size` and `value_size` fields defined how much memory was required for them, but they were just treated as so many bytes. By additionally passing in the BTF information defining the types of the keys and values, the kernel can introspect them, and utilities like `bpftool` can retrieve the type information for pretty-printing, as discussed earlier. However, it’s interesting to note that separate BTF `type _id`s are passed in for the key and the value. The `____btf_map_config` structure that you just saw defined isn’t used by the kernel for the map definition; it’s just used by BCC on the user space side.