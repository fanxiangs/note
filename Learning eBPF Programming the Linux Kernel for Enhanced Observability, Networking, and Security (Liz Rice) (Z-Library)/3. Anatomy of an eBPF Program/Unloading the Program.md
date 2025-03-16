# Unloading the Program

[[null|]][[null|]][[null|]][[null|]][[null|]]There’s no inverse of `bpftool prog load` (at least not at the time of this writing), but you can remove the program from the kernel by deleting the pinned pseudofile:

$ rm /sys/fs/bpf/hello
$ bpftool prog show name hello

There is no output from this `bpftool` command because the program is no longer loaded in the kernel.