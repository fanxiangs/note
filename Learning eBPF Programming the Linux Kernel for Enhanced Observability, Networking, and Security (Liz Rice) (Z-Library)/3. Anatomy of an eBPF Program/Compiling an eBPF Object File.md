# Compiling an eBPF Object File

[[null|]][[null|]]Our eBPF source code needs to be compiled into the machine instructions that the eBPF virtual machine can understand: eBPF bytecode. [[null|]]The Clang compiler from the [LLVM project](https://llvm.org) will do this if you specify `-target bpf`. The following is an extract from a Makefile that will do the compilation:

    hello.bpf.o

This generates an object file called _hello.bpf.o_ from the source code in _hello.bpf.c_. The `-g` flag is optional here,[^3] but it generates debug information so that you can see the source code alongside the bytecode when you inspect the object file. Let’s inspect this object file to better understand the eBPF code it contains.