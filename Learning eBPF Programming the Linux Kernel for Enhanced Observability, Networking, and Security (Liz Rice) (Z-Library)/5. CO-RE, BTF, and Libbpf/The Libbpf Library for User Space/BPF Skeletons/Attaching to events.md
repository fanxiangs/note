# Attaching to events

[[null|]][[null|]]The next skeleton function in the example attaches the program to the `execve` syscall function:

    err

The _libbpf_ library automatically takes the attachment point from the `SEC()` definition for this program. If you didn’t define the attachment point fully, there are a whole series of _libbpf_ functions, such as `bpf_program__attach_kprobe`, `bpf_program__attach_xdp`, and so on, for attaching different program types.