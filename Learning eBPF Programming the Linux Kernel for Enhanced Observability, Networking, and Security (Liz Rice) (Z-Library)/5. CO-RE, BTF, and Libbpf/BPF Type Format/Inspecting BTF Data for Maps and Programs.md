# Inspecting BTF Data for Maps and Programs

[[null|]][[null|]][[null|]][[null|]]If you want to inspect the BTF types associated with a particular map, `bpftool` makes that easy. For example, here’s the output for the `config` map:

bpftool btf dump map name config
\[1\] TYPEDEF 'u32' type\_id=2
\[4\] STRUCT 'user\_msg\_t' size=12 vlen=1
        'message' type\_id=6 bits\_offset=0

Similarly, you can inspect the BTF information related to a particular program with `bpftool btf dump prog` . I’ll leave you to check out the [manpage](https://oreil.ly/lCoV5) for additional details.

##### Note

If you’d like to better understand how the BTF type data is generated and de-duplicated, there is another [excellent blog post from Andrii Nakryiko](https://oreil.ly/0-a9g) on the subject.

By this stage you should have an understanding of how BTF describes the format of data structures and functions. An eBPF program written in C needs header files that define the types and structures. Let’s see how easy it is to generate a header file for any kernel data types that an eBPF program might need.[[null|]]