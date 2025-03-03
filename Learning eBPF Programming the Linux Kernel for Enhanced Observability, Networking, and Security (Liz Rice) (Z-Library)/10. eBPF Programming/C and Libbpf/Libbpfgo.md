# Libbpfgo

[[null|]]The [libbpfgo project](https://oreil.ly/gvbXr) by Aqua Security implements a Go wrapper around _libbpf_’s C code, providing utilities for loading and attaching programs and using Go-native features like channels for receiving events. Because it’s built on _libbpf_, it supports CO-RE.

Here’s an extract from the example from _libbpfgo_’s _README_, which gives a good high-level view of what to expect from this library:

    bpfModule

[[#code_id_10_11|]]

Read eBPF bytecode from an object file.

[[#code_id_10_12|]]

Load that bytecode into the kernel.

[[#code_id_10_13|]]

Manipulate an entry in an eBPF map.

[[#code_id_10_14|]]

Go programmers will appreciate receiving data from a ring or perf buffer on a channel, which is a language feature designed to handle asynchronous events.

This library was created for Aqua’s [Tracee](https://oreil.ly/A03zd) security project, and it’s also being used by other projects such as [Parca](https://oreil.ly/s8JP9) from Polar Signals, which provides eBPF-based CPU profiling. The only concern about this project’s approach is the CGo boundary between the _libbpf_ C code and Go, which can cause performance and other issues.[^5]

While Go has been the established language for lots of infrastructure coding for around a decade, there has more recently been a growing body of developers who prefer to use Rust.[[null|]][[null|]][[null|]]