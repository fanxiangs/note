# Loading the Program into the Kernel

[[null|]][[null|]][[null|]]For this example we’ll use a utility called `bpftool`. You can also load programs programmatically, and you’ll see examples of that later in the book.

###### Note

Some Linux distributions provide a package that includes `bpftool`, or you can [compile it from source code](https://github.com/libbpf/bpftool). You can find more details about installing or building this tool on [Quentin Monnet’s blog](https://oreil.ly/Yqepv), as well as additional documentation and usage on the [Cilium site](https://oreil.ly/rnTIg).

The following is an example of using `bpftool` to load a program into the kernel. [[null|]]Note that you’ll likely need to be root (or use `sudo`) to get the BPF privileges that `bpftool` requires.

$ bpftool prog load hello.bpf.o /sys/fs/bpf/hello

This loads the eBPF program from our compiled object file and “pins” it to the location _/sys/fs/bpf/hello_.[^4] No output response to this command indicates success, but you can confirm that the program is in place using `ls`:

$ ls /sys/fs/bpf
hello

The eBPF program has been successfully loaded. Let’s use the `bpftool` utility to find out more about the program and its status within the kernel.