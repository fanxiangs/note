# From BPF to eBPF

[[null|]][[null|]]BPF evolved to what we call “extended BPF” or “eBPF” starting in kernel version 3.18 in 2014. This involved several significant changes:

*   The BPF instruction set was completely overhauled to be more efficient on 64-bit machines, and the interpreter was entirely rewritten.
*   eBPF _maps_ were introduced, which are data structures that can be accessed by BPF programs and by user space applications, allowing information to be shared between them. You’ll learn about maps in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]].
*   The `bpf()` system call was added so that user space programs can interact with eBPF programs in the kernel. You’ll read about this system call in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]].
*   Several BPF helper functions were added. You’ll see a few examples in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]] and some more details in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]].
*   The eBPF verifier was added to ensure that eBPF programs are safe to run. This is discussed in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]].

This put the basis for eBPF in place, but development did not slow down! Since then, eBPF has evolved significantly.