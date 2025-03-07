# The Libbpf Library for User Space

[[null|]][[null|]]The _libbpf_ library is a user space library you can use directly if you’re writing the user space part of your application in C. If you want to, you can use this library without using CO-RE. There’s an example of this in [Andrii Nakryiko’s excellent blog post on libbpf-bootstrap](https://oreil.ly/b3v7B).

[[null|]]This library provides functions that wrap the `bpf()` and related syscalls that you met in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]] to perform operations like loading programs into the kernel and attaching them to events, or accessing map information from user space. The conventional and easiest way to use these abstractions is through auto-generated BPF skeleton code.