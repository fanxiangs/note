# Makefile

[[null|]][[null|]]The following is an example Makefile instruction for compiling CO-RE objects (taken from the Makefile in the _chapter5_ directory of the GitHub repo for this book):

    hello-buffer-config.bpf.o

If you’re using the example code, you should be able to build the eBPF object file _hello-buffer-config.bpf.o_ (and its companion user space executable that I’ll describe shortly) by running `make` in the _chapter5_ directory. Let’s inspect that object file to see that it includes BTF information.