# BTF Use Cases

[[null|]]The main reason for discussing BTF in this chapter on CO-RE is that knowing the differences between a structure’s layout where an eBPF program was compiled and where it is about to run allows for the appropriate adjustments to be made as the program is loaded into the kernel. I’ll discuss the relocation process later in this chapter, but for now, let’s also consider some of the other uses to which BTF information can be put.

Knowing how a structure is laid out, and the type of every field in that structure, makes it possible to pretty-print a structure’s contents in human-readable form. For example, a string is just a series of bytes from the computer’s point of view, but converting those bytes into characters makes the string much easier for humans to understand. You already saw an example of this in the previous chapter, where `bpftool` used BTF information to format the output of map dumps.

BTF information also includes the line and function information that enables `bpftool` to interleave source code within the output from translated or JITed program dumps, as you saw in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]. When you come to [[ch06.xhtml#the_ebpf_verifier|Chapter 6]], you’ll also see the source code information interleaved with the verifier log output, and again this comes from the BTF information.

[[null|]]BTF information is also required for BPF spin locks. _Spin locks_ are used to stop two CPU cores from simultaneously accessing the same map values. The lock has to be part of the map’s value structure, like this:

struct my\_value {
     ... 
     struct bpf\_spin\_lock lock;
... 
};

Within the kernel, eBPF programs use `bpf_spin_lock()` and `bpf_spin_unlock()` helper functions to acquire and release a lock. These helpers can be used only if BTF information is available to describe where the lock field is within the structure.

##### Note

Spin lock support was added in kernel version 5.1. There are lots of restrictions on the use of spin locks: they can only be used on hash or array map types, and they can’t be used in tracing or socket filter type eBPF programs. Read more about spin locks in the [lwn.net article on concurrency management in BPF](https://oreil.ly/kAyAU).

Now that you know why BTF information is useful, let’s make it more concrete by looking at some examples.