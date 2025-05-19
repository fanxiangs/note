# Accessing existing maps

[[null|]][[null|]][[null|]]By default, _libbpf_ will also create any maps that are defined in the ELF bytes, but sometimes you might want to write an eBPF program that reuses an existing map. You already saw an example of this in the previous chapter, where you saw `bpftool` iterating through all the maps, looking for the one that matched a specified name. Another common reason to use a map is to share information between two different eBPF programs, so only one program should create the map. The `bpf_map__set_autocreate()` function allows you to override _libbpf_’s auto-creation.

So how do you access an existing map? Maps can be pinned, and if you know the pinned path, you can get a file descriptor to an existing map with `bpf_obj_get()`. Here’s a very simple example (available in the GitHub repository as _chapter5/find-map.c_):

    struct

To try this out you can create a map using `bpftool`, like this:

$ bpftool map create /sys/fs/bpf/findme type array key 4 value 32 entries 4
name findme

Running the find-map executable will print out:

Name: findme

Let’s get back to the _hello-buffer-config_ example and the skeleton code.