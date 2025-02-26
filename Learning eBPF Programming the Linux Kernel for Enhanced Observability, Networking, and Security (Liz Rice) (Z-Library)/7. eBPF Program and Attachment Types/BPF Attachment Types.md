# BPF Attachment Types

[[null|]][[null|]]The attachment type offers more fine-grained control over where a program can be attached in the system. For some program types there is a one-to-one correlation to the type of hook that it can be attached to, so the attachment type is implicitly defined by the program type. For example, XDP programs are attached to XDP hooks in the network stack. For a few program types, an attachment type also has to be specified.

The attachment type is involved in deciding which helper functions are valid, and it also restricts access to parts of the context information in some cases. There was an example of this earlier in this chapter where the verifier gives an `invalid bpf_context access` error.

You can also see which program types need an attachment type to be specified, and which attachment types are valid, in the kernel function [bpf_prog_load_check_attach](https://oreil.ly/0LqCQ) (defined in [bpf/syscall.c](https://oreil.ly/7OrYS)).

For example, here is the code that checks the attachment type for a program of type `CGROUP_SOCK`:

    case

This program type can be attached in multiple places: at socket creation, at socket release, or after a bind is completed in IPv4 or IPv6.

Another place to find a listing of the valid attachment types for programs is the [libbpf documentation](https://oreil.ly/jraLh), where you’ll also find the section names that _libbpf_ understands for each program and attachment type.