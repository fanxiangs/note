# BCC Python/Lua/C++

[[null|]][[null|]][[null|]]Back in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]], the first “Hello World” example I gave you was a Python program written using the BCC library. This project includes plenty of useful performance measurement tools implemented using this same library (as well as newer implementations based on _libbpf_ that I’ll come to momentarily).

In addition to the [documentation](https://oreil.ly/Elggv) that describes how to use the provided BCC tools to measure performance, BCC also includes a [reference guide](https://oreil.ly/WgeJA) and a [Python programming tutorial](https://oreil.ly/hR3xr) to help you develop your own eBPF tools in this framework.

[[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]] included a discussion of BCC’s approach to portability, which is to compile the eBPF code at runtime to ensure that it’s compatible with the target machine’s kernel data structures. In BCC, you define the kernel-side eBPF program code as a string (or the contents of a file that BCC reads into a string). This string gets passed to Clang for compilation, but before that happens, BCC does some preprocessing on the string. This enables it to provide handy shortcuts for the programmer, some of which you’ve seen already in this book. For example, here are some relevant lines from the example code in _chapter2/hello\_map.py_:

    #!/usr/bin/python3                                

[[#code_id_10_1|]]

This is a Python program, which will run in user space.

[[#code_id_10_2|]]

The `program` string holds the eBPF program to be compiled and then loaded into the kernel.

[[#code_id_10_3|]]

`BPF_RINGBUF_OUTPUT` is a BCC macro that defines a ring buffer called `output`. This is part of the `program` string, so it’s natural to assume it’s defining the buffer from the kernel’s perspective. Hold that thought until we get to callout 6.

[[#code_id_10_4|]]

This line looks like a `ringbuf_output()` method on an object called `object`. But wait a minute—methods on objects aren’t even part of the C language! BCC is doing some heavy lifting here, [expanding methods](https://oreil.ly/vLVth) like these into underlying BPF helper functions, `bpf_ringbuf_output()` in this case.

[[#code_id_10_5|]]

This is where the program string gets rewritten into the BPF C code that Clang can compile. This line also loads the resultant program into the kernel.

[[#code_id_10_6|]]

There is no other place in the code that defines the ring buffer called `output`, and yet it’s accessible from the Python user space code here. BCC does double duty when it preprocesses the line in callout 3, as it defines the ring buffer for both the user space and kernel parts.

As this example shows, BCC essentially provides its own C-like language for BPF programming. It makes life easy for the programmer, handling things like shared structure definitions for both kernel and user space and providing convenient shortcuts to wrap around BPF helper functions. This means BCC is an accessible way to get into eBPF programming if you’re new to the field, especially if you’re already comfortable with Python.

###### Note

If you’d like to explore BCC programming, this [tutorial aimed at Python programmers](https://oreil.ly/0pHKY) is a great way to walk through many more of the features and capabilities of BCC than there is space for in this book.

The documentation doesn’t make it terribly clear, but as well as supporting Python as the language for the user space part of eBPF tools, [[null|]][[null|]]BCC enables writing tools in Lua and C++. There are _lua_ and _cpp_ directories within the supplied [examples](https://oreil.ly/PP0cL) that you can base your own code on, should you be keen to try this approach.

BCC may be convenient for the programmer, but because of the inefficiencies of distributing a compiler toolchain alongside your utility (discussed in more depth in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]]), if you’re looking to write production-quality tools that you intend to distribute, I recommend considering some of the other libraries discussed in this chapter.[[null|]][[null|]][[null|]]