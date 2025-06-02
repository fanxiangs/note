# Debug Information

[[null|]][[null|]]You have to pass the `-g` flag to Clang so that it includes debug information, which is necessary for BTF. However, the `-g` flag also adds DWARF debugging information to the output object file, but that’s not needed by eBPF programs, so you can reduce the size of the object by running the following command to strip it out:

llvm-strip -g