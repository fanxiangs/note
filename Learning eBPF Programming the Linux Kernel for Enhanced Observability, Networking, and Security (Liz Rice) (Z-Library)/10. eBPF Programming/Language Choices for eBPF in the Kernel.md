# Language Choices for eBPF in the Kernel

[[null|]]eBPF programs can be written directly in eBPF bytecode,[^2] but in practice, most are compiled to bytecode from either C or Rust. These languages have compilers that support eBPF bytecode as a target output.

###### Note

eBPF bytecode isn’t a suitable target for all compiled languages. If the language involves a runtime component (like Go, or Java’s virtual machine), it’s likely to be incompatible with eBPF’s verifier. For example, it’s hard to imagine how memory garbage collection could work hand in hand with the verifier’s checks on safe use of memory. Similarly, eBPF programs are required to be single threaded, so any concurrency features in a language couldn’t be used.

Although not really eBPF, there is an interesting project called [XDPLua](https://oreil.ly/7_3Fx) that proposes writing XDP programs in Lua scripts that run directly within the kernel. However, the initial research in this project suggested that eBPF would likely be more performant, and as eBPF becomes more powerful with each kernel release (e.g., now being able to implement loops), it’s not clear that there is much advantage other than the personal preference that some people might have to write code in Lua scripts.

I would hazard a guess that most people who choose to write eBPF kernel code in Rust would also opt for the same language for the user space code, since shared data structures wouldn’t need to be rewritten. It’s not obligatory, though—you can mix and match eBPF code with whatever user space language you choose.

[[null|]]Those who choose to write the kernel-side code in C also have the option to write user space code in C (you’ve seen plenty of examples of that in this book already). But C is a pretty low-level language that requires programmers to handle lots of details for themselves, notably, memory management. While some people are comfortable doing this, many people would prefer to write the user space code in another, higher-level language. Whatever your preferred language, you’d like a library that provides eBPF support so that you don’t have to write directly to the system call interface you saw in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]. In the rest of this chapter we’ll discuss some of the most popular options for eBPF libraries in a variety of languages.