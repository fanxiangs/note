# Function Calls

[[null|]][[null|]]You’ve seen that eBPF programs can call helper functions provided by the kernel, but what if you want to split the code you’re writing into functions? Generally, in software development it’s considered good practice[^8] to pull common code into a function that you can call from multiple places, rather than duplicating the same lines over and over again. But in the early days, eBPF programs were not permitted to call functions other than helper functions. To work around this, programmers have directed the compiler to “always inline” their functions, like this:

    static

[[null|]]Generally, a function in source code results in the compiler emitting a jump instruction, which causes execution to jump to the set of instructions that make up the called function (and then to jump back again when that function has completed). You can see this illustrated on the left side of [[#layout_of_noninlined_and_inlined_functi|Figure 2-5]]. The right side shows what happens when a function is inlined: there is no jump instruction; instead, a copy of the function’s instructions is emitted directly within the calling function.

![Layout of noninlined and inlined function instructions](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0205.png)

##### Figure 2-5. Layout of noninlined and inlined function instructions

If the function is called from multiple places, that results in multiple copies of that function’s instructions in the compiled executable. (Sometimes the compiler might choose to inline a function for optimization purposes, and that is one reason why you might not be able to attach a kprobe to certain kernel functions. I’ll come back to this in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].)

[[null|]][[null|]]Starting from Linux kernel 4.16 and LLVM 6.0, the restriction requiring functions to be inlined was lifted so that eBPF programmers could write function calls more naturally. However, this feature, called “BPF to BPF function calls” or “BPF subprograms,” isn’t currently supported by the BCC framework, so let’s come back to it in the next chapter. (You can, of course, continue to use functions with BCC if they are inlined.)

There is another mechanism for decomposing complex functionality into smaller parts in eBPF: tail calls.