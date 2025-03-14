# Libbpf-rs

[[null|]][[null|]][Libbpf-rs](https://oreil.ly/qBagk) is part of the _libbpf_ project, and provides a Rust wrapper around the _libbpf_ C code so that you can write the user space parts of eBPF code in Rust. As you can see from the project’s [examples](https://oreil.ly/6wpf8), the eBPF programs themselves are written in C.

##### Note

There are further examples in Rust in the [libbpf-bootstrap](https://oreil.ly/ter6c) project, designed to help you get off the ground if you want to try building your own code using this crate.

This crate is helpful for incorporating eBPF programs into a Rust-based project, but it doesn’t fulfill the desire that many people have to write the kernel-side code in Rust as well. Let’s look at some other projects that enable that.