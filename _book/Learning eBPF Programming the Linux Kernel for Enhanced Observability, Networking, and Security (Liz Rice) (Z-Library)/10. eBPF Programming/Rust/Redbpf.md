# Redbpf

[[null|]][[null|]][Redbpf](https://oreil.ly/AtJod) is a set of Rust crates that interface with _libbpf_, developed as part of [foniod](https://oreil.ly/dwGNK), an eBPF-based security monitoring agent.

[[null|]]_Redbpf_ predates Rust’s ability to compile to eBPF bytecode, so it uses a [multistep compilation process](https://oreil.ly/DuHxE) that involves compiling from Rust to LLVM bitcode and then using the LLVM toolchain to generate eBPF bytecode in ELF format. _Redbpf_ supports a range of program types including tracepoints, kprobes and uprobes, XDP, TC, and some socket events.

As the Rust compiler rustc gained the ability to generate eBPF bytecode directly, this was leveraged by a project called Aya. At the time of this writing, Aya is considered “emerging” according to the [community site at ebpf.io](https://oreil.ly/WynV6), while _Redbpf_ is listed as a major project, but my personal perspective is that momentum seems to be moving toward Aya.