# Summary

The vast majority of people who use eBPF-based tooling won’t need to write eBPF code themselves, but if you do find yourself wanting to implement something yourself, you have a lot of options. This is a changing field, so it’s very possible that by the time you read this, new language libraries and frameworks might exist, or consensus may have gathered around some of the libraries I’ve highlighted in this chapter. You’ll find an up-to-date list of the major language projects around eBPF on the [Infrastructure page of ebpf.io’s list of significant projects](https://ebpf.io/infrastructure).

For quickly collecting trace information, `bpftrace` can be a very valuable option.

For more flexibility and control, BCC is a fast way to build an eBPF tool if you’re comfortable with Python, provided that you don’t care about the compilation step that takes place at runtime.

If you’re writing eBPF code to be widely distributed and portable across different kernel versions, you’ll probably want to take advantage of CO-RE. The user space frameworks that support CO-RE at time of this writing are _libbpf_ for C, _cilium/ebpf_ and _libbpfgo_ for Go, and Aya for Rust.

For further advice, I highly recommend joining the [eBPF Slack](http://ebpf.io/slack) and discussing your questions there. You’ll likely find the maintainers of many of these language libraries in that community.[[null|]]