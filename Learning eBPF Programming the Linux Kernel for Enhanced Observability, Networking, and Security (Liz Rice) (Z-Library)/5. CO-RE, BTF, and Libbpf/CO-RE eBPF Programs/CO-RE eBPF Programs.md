# CO-RE eBPF Programs

[[null|]]You’ll recall that eBPF programs run in the kernel. Later in this chapter I’ll show some user space code that will interact with the code running in the kernel, but in this section I’m concentrating on the kernel side.

As you’ve already seen, eBPF programs are compiled to eBPF bytecode, and (at least at the time of this writing) the compilers that support this are Clang or gcc for compiling C code, and the Rust compiler. I’ll discuss some of your options for using Rust in [[ch10.xhtml#ebpf_programming|Chapter 10]], but for the purposes of this chapter I’ll assume you’re writing in C and using Clang, along with the _libbpf_ library.

For the remainder of this chapter, let’s consider an example application called _hello-buffer-config_. It’s very similar to the _hello-buffer-config.py_ example from the previous chapter that used the BCC framework, but this version is written in C to use _libbpf_ and CO-RE.

If you have BCC-based eBPF code that you want to migrate to _libbpf_, check out the excellent and comprehensive [guide by Andrii Nakryiko on his website](https://oreil.ly/iWDcv). BCC provides some convenient shortcuts that aren’t handled in quite the same way using _libbpf_; conversely, _libbpf_ provides its own set of macros and library functions to make life easier for the eBPF programmer. As I walk through the example, I will point out a few differences between the BCC and _libbpf_ approaches.

###### Note

You’ll find the example C eBPF program to accompany this section in the _chapter5_ directory of the [github.com/lizrice/learning-ebpf](https://github.com/lizrice/learning-ebpf) repo.

First let’s look at _hello-buffer-config.bpf.c_, which implements the eBPF program that runs in the kernel. Later in the chapter I’ll show you the user space code in _hello-buffer-config.c_ that loads the program and displays output, much as the Python code did in the BCC implementation of this example in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]].

Like any C program, an eBPF program will need to include some header files.