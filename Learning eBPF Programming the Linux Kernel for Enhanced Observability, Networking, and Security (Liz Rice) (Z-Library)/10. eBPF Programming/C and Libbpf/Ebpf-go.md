# Ebpf-go

[[null|]][[null|]][[null|]]The [eBPF Go library included as part of the Cilium project](https://oreil.ly/BnGyl) is widely used (I found around 10,000 references on GitHub, and the project has close to 4,000 stars). It provides convenient functions for managing and loading eBPF programs and maps, including CO-RE support, all implemented purely in Go.

With this library you have the option to compile your eBPF programs to bytecode and embed that bytecode into Go source code, using a supplied tool called [bpf2go](https://oreil.ly/-kDbH). You need the LLVM/Clang compiler to do this generation as part of the build step. Once the Go code is compiled, you have a single Go binary that you can distribute that includes the eBPF bytecode and is portable to different kernels, without any dependencies other than the Linux kernel itself.

The _cilium/ebpf_ library also supports loading and managing eBPF programs built as standalone ELF files (like the _\*.bpf.o_ examples you have seen in this book).

At the time of this writing, the _cilium/ebpf_ library supports the perf events for tracing, including the relatively recent fentry events, as well as an extensive set of network program types like XDP and cgroup socket attachments.

In this project’s [examples directory under cilium/ebpf](https://oreil.ly/Vuf9d), you’ll see that the C code for in-kernel programs sits in the same directories as the corresponding user space code in Go:

*   The C files start with `// +build ignore`, which tells the Go compiler to ignore them. At the time of this writing there is an [update in progress](https://oreil.ly/ymuyn) to change to the newer `//go:build` style of build tag.
    
*   The user space files include a line like the following, which tells the Go compiler to invoke the bpf2go tool on the C file(s):
    
    //go:generate go run github.com/cilium/ebpf/cmd/bpf2go -cc $BPF\_CLANG
                         -cflags $BPF\_CFLAGS bpf  -- -I../headers
    
    Running `go:generate` on the package rebuilds the eBPF program and regenerates the skeleton in a single step.
    

Much like `bpftool gen skeleton`, which you saw in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], `bpf2go` generates skeleton code for manipulating the eBPF objects, minimizing the user space code you need to write yourself (except it’s generating Go code rather than C). The output files also include the _.o_ object files containing the bytecode.

In fact, `bpf2go` generates two versions of the bytecode _.o_ files, for big- and little-endian architectures. There are also two correspondingly generated _.go_ files, and the correct versions for the target platform get used at compile time. As an example, the auto-generated files in the [kprobe example from cilium/ebpf](https://oreil.ly/CgwVd) are:

*   The _bpf\_bpfeb.o_ and _bpf\_bpfel.o_ ELF files containing eBPF bytecode
    
*   The _bpf\_bpfeb.go_ and _bpf\_bpfel.go_ files, which define Go structures and functions that correspond to the maps, programs, and links defined in that bytecode
    

You can relate the objects defined in the auto-generated Go code to the C code from which it was generated. Here are the objects defined in the C code for that kprobe example:

    struct

The auto-generated Go code includes structures representing all the maps and programs (in this case, there is only one of each):

    type

The names “KprobeMap” and “KprobeExecve” are derived from the map and program names used in the C code. These objects are grouped into a `bpfObjects` structure representing everything that’s being loaded into the kernel:

    type

You can then use these object definitions and related auto-generated functions in your user space Go code. To give you an idea of what this might involve, here’s an extract based on the main function from the same [kprobe example](https://oreil.ly/YXAjH) (omitting error handling for brevity):

    objs

[[#code_id_10_7|]]

Load all the BPF objects that were embedded in bytecode form, into the `bpfObjects` I just showed you defined by the auto-generated code.

[[#code_id_10_8|]]

Attach the program to the `sys_execve` kprobe.

[[#code_id_10_9|]]

Set up a ticker so that the code can poll the map once per second.

[[#code_id_10_10|]]

Read an item out of the map.

There are several other examples in the _cilium/ebpf_ directory that you can use for reference and inspiration.[[null|]][[null|]][[null|]]