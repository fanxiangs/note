# Reading Information from a Map

[[null|]][[null|]][[null|]]The following command shows an extract of the `bpf()` syscalls that `bpftool` makes while reading the contents of the `config` map:

$ strace -e bpf bpftool map dump name config

As you’ll see, the sequence consists of two main steps:

*   Iterate through all the maps, looking for any with the name `config`.
    
*   If a matching map is found, iterate through all the elements in that map.