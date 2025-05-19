# Aya

[[null|]][[null|]][Aya](https://aya-rs.dev/book) is built in Rust directly to the syscall level, so it doesn’t depend on _libbpf_ (or indeed on BCC or the LLVM toolchain). But it does support the BTF format, the same relocations that _libbpf_ does (as described in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]]), so it’s providing the same CO-RE abilities to compile once and run on other kernels. At the time of this writing, it supports a wider range of eBPF program types than _Redbpf_, including tracing/perf-related events, XDP and TC, cgroups, and LSM attachments.

As I mentioned, the Rust compiler also supports [compiling to eBPF bytecode](https://oreil.ly/a5q7M), so this language can be used for both kernel and user space eBPF programming.

##### Note

The ability to write both the kernel side and the user space side natively in Rust without the intermediate dependency on LLVM has attracted Rust programmers to this option. There’s an interesting [discussion on GitHub](https://oreil.ly/nls4l) about why the developers of the [lockc project](https://oreil.ly/_-L6z) (an eBPF-based project that enhances the security of container workloads using LSM hooks) decided to port their project from _libbpf-rs_ to Aya.

The project includes [aya-tool](https://oreil.ly/Kd0nf), a utility for generating Rust structure definitions that match kernel data structures so that you don’t have to write them yourself.

The Aya project strongly emphasizes developer experience and makes it easy for newcomers to get started. With that in mind, the [“Aya book”](https://aya-rs.dev/book) is a very readable introduction with some good example code, annotated with helpful explanations.

To give you a brief idea of what eBPF code looks like in Rust, here’s an extract from Aya’s basic XDP example that permits all traffic:

    #[

[[#code_id_10_15|]]

This line is what defines the section name, equivalent to `SEC("xdp/myapp")` in C.

[[#code_id_10_16|]]

The eBPF program called `myapp` calls the function `try_myapp` to process a network packet received at XDP.

[[#code_id_10_17|]]

The `try_myapp` function logs the fact that a packet was received and always returns the `XDP_PASS` value that tells the kernel to carry on processing the packet as usual.

Just as we’ve seen in C-based examples throughout this book, the eBPF program gets compiled to an ELF object file. The difference is that Aya uses the Rust compiler instead of Clang to create that file.

Aya also generates code for the user space activities of loading the eBPF program into the kernel and attaching it to an event. Here are a few key lines from the user space side of that same basic example:

    let

[[#code_id_10_19|]]

Read the eBPF bytecode from the ELF object file produced by the compiler.

[[#code_id_10_20|]]

Find the program called `myapp` in that bytecode.

[[#code_id_10_21|]]

Load it into the kernel.

[[#code_id_10_22|]]

Attach it to the XDP event on a specified network interface.

If you’re a Rust programmer, I highly recommend you explore the [additional examples](https://oreil.ly/bp_Hq) in the “Aya book” in more detail. There’s also a nice [blog post from Kong](https://oreil.ly/mUVIk) that walks through writing an XDP load balancer using Aya.

##### Note

Aya maintainers Dave Tucker and Alessandro Decina joined me for [episode 25 of the “eBPF and Cilium Office Hours” livestream](https://oreil.ly/U7bRu) where they demonstrated and gave an introduction to eBPF programming with Aya.[[null|]][[null|]]