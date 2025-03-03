# Rust

[[null|]][[null|]]Rust is increasingly being used for building infrastructure tools. It allows for the low-level access of C, but with the added benefit of memory safety. Indeed, Linus Torvalds [confirmed in 2022](https://oreil.ly/7fINA) that the Linux kernel itself will start to incorporate Rust code, and the recent [6.1 release has some initial Rust support](https://oreil.ly/HrXy2).

As I discussed earlier in this chapter, Rust can be compiled to eBPF bytecode, meaning that (with the right library support) it’s possible to write both the user space and kernel code for eBPF utilities in Rust.

There are a few options for Rust eBPF development: _libbpf-rs_, _Redbpf_, and Aya.