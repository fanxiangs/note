# Memory Access with CO-RE

[[null|]][[null|]]eBPF programs for tracing have restricted access to memory, via a BPF helper function from the `bpf_probe_read_*()` family.[^8] (There is also a `bpf_probe_write_user()` helper function, but it’s only [“meant for experiments”](https://oreil.ly/ibcy1)). The problem is that, as you’ll see in the next chapter, the eBPF verifier generally won’t let you simply read memory through a pointer as you usually can in C (e.g., `x = p->y`).[^9]

The _libbpf_ library provides CO-RE wrappers around the `bpf_probe_read_*()` helpers to take advantage of the BTF information and make memory access calls portable across different kernel versions. Here’s an example of one of those wrappers, as defined in the [bpf_core_read.h header file](https://oreil.ly/XWWyc):

    #define bpf_core_read(dst, sz, src)                        \

As you can see, `bpf_core_read()` calls directly to `bpf_probe_read_kernel()`, the only difference being that it wraps the `src` field with `__builtin_preserve_access_index()`. [[null|]]This tells Clang to emit a CO-RE relocation entry along with the eBPF instruction that accesses this address in memory.

##### Note

This `__builtin_preserve_access_index()` instruction is an extension to “regular” C code, and adding it to eBPF also required changes to the Clang compiler to support it and emit these CO-RE relocation entries. Extensions like these are examples of why some C compilers cannot (today, at least) generate eBPF bytecode. Read more about the Clang changes required for eBPF CO-RE support on the [LLVM mailing list](https://oreil.ly/jHTHE).

As you’ll see later in this chapter, the CO-RE relocation entry tells _libbpf_ to rewrite the address, as it’s loading the eBPF program into the kernel, to take account of any BTF differences. If the offset of `src` within its containing structure is different on the target kernel, the rewritten instruction will take that into account.

The _libbpf_ library provides a `BPF_CORE_READ()` macro so that you can write several `bpf_core_read()` calls in a single line rather than needing a separate helper function call for every pointer dereference. For example, if you wanted to do something like `d = a->b->c->d`, you could write the following code:

    struct

But it’s much more compact to use:

    d

You can then read from point `d` using the `bpf_probe_read_kernel()` helper function.

There’s a good description of this in Andrii’s [guide](https://oreil.ly/tU0Gb).