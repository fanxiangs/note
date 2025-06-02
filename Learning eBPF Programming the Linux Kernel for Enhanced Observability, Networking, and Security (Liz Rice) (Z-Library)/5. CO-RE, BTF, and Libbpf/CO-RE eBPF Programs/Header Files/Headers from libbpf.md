# Headers from libbpf

[[null|]][[null|]][[null|]]To use any BPF helper functions in your eBPF code, you’ll need to include the header files from _libbpf_ that give you their definitions.

#### Note

One thing that can be slightly confusing about _libbpf_ is that it’s not just a user space library. You’ll find yourself including header files from _libbpf_ in both user space and eBPF C code.

At the time of this writing, it is common to see eBPF projects including _libbpf_ as a submodule and building/installing from source—this is what I have done in the example repository for this book. If you include it as a submodule, you’ll simply need to run `make install` from the _libbpf/src_ directory. I don’t think it will be long before it’s more common to see _libbpf_ widely available as a package on common Linux distributions, particularly since _libbpf_ has now passed the milestone of a [version 1.0 release](https://oreil.ly/8BFq6).