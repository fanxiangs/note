# Generating a Kernel Header File

[[null|]][[null|]][[null|]][[null|]]If you run `bpftool btf list` on a BTF-enabled kernel, you’ll see lots of preexisting blobs of BTF data that look like this:

$ bpftool btf list
1: name \[vmlinux\]  size 5842973B
2: name \[aes\_ce\_cipher\]  size 407B
3: name \[cryptd\]  size 3372B
...

The first item in this list, with ID 1 and named `vmlinux`, is the BTF information about all the data types, structures, and function definitions used by the kernel that’s running on this (virtual) machine.[^5]

An eBPF program needs the definitions of any kernel data structures and types that it is going to refer to. Before the days of CO-RE, you’d typically have to figure out which of the many individual header files in the Linux kernel source held the definition for the structures you were interested in, but now there is a much easier way, as BTF-enabled tools can generate an appropriate header file from the BTF information included with the kernel.

[[null|]]This header file is conventionally called _vmlinux.h_, and you can generate it with `bpftool` like this:

bpftool btf dump file /sys/kernel/btf/vmlinux format c > vmlinux.h

This file defines all the kernel’s data types, so including this generated _vmlinux.h_ file in your eBPF program source supplies the definitions of any Linux data structures you might need. When you compile the source into an eBPF object file, that object will include BTF information that matches the definitions used in this header file. Later, when the program is run on a target machine, the user space program that loads it into the kernel will make adjustments to account for differences between this build-time BTF information and the BTF information for the kernel that’s running on that target machine.

BTF information in the form of the _/sys/kernel/btf/vmlinux_ file has been included in the Linux kernel since version 5.4,[^6] but raw BTF data that _libbpf_ can make use of can also be generated for older kernels. In other words, if you want to run a CO-RE–enabled eBPF program on a target machine that doesn’t have BTF information already, you might be able to provide the BTF data for that target yourself. There’s information on how to generate BTF files, and an archive of files for a variety of Linux distributions, on the [BTFHub](https://oreil.ly/mPSO0).

###### Note

The BTFHub repo also includes further reading about [BTF internals](https://oreil.ly/CfyQh) should you want to dive deeper into this topic.[[null|]][[null|]][[null|]][[null|]]

Next, let’s look at how this and other tactics are used to write eBPF programs to be portable across kernels using CO-RE.