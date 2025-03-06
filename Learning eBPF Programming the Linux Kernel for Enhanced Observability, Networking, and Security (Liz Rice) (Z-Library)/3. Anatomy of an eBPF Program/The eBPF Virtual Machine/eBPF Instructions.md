# eBPF Instructions

[[null|]][[null|]]The same [linux/bpf.h header file](https://oreil.ly/_ZhU2) defines a structure called `bpf_insn`, which represents a BPF instruction:

```c
struct bpf_insn {
    __u8 code;          /* opcode */                       
    __u8 dst_reg:4;     /* dest register */               
    __u8 src_reg:4;     /* source register */
    __s16 off;       /* signed offset */                  
    __s32 imm;       /* signed immediate constant */
};
```
1. Each instruction has an opcode, which defines what operation the instruction is to perform: for example, adding a value to the contents of a register, or jumping to a different instruction in the program.[^2] The Iovisor project’s [“Unofficial eBPF spec”](https://oreil.ly/FXcPu) has a list of the valid instructions.
2. Different operations might involve up to two registers.
3. Depending on the operation, there might be an offset value and/or an “immediate” integer value.

This `bpf_insn` structure is 64 bits (or 8 bytes) long. However, sometimes an instruction might need to span more than 8 bytes. [[null|]]If you want to set a register to a 64-bit value, you can’t somehow squeeze all 64 bits of that value into the structure, along with the opcode and register information. In these cases, the instruction uses _wide instruction encoding_ that is 16 bytes long in total. You’ll see an example of this in this chapter.

When loaded into the kernel, the bytecode of an eBPF program is represented by a series of these `bpf_insn` structures. The verifier performs several checks on this information to ensure that the code is safe to run. You’ll learn more about the verification process in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]].

Most of the different opcodes fall into the following categories:

*   Loading a value into a register (either an immediate value or a value read from memory or from another register)
*   Storing a value from a register into memory
*   Performing arithmetic operations such as adding a value to the contents of a register
*   Jumping to a different instruction if a particular condition is satisfied

> [!Note]
For an overview of eBPF architecture, I recommend the [BPF and XDP Reference Guide](https://oreil.ly/rvm1i) that’s included as part of the Cilium project’s documentation. If you’d like more details, the [kernel documentation](https://oreil.ly/_2XDT) describes the eBPF instructions and encoding quite clearly.

Let’s use another simple example of an eBPF program and follow its journey from C source code, through eBPF bytecode, to machine code instructions.

> [!Note]
If you want to build and run this code yourself, you’ll find the code along with instructions for setting up an environment to do so at [github.com/lizrice/learning-ebpf](https://github.com/lizrice/learning-ebpf). The code for this chapter is in the _chapter3_ directory.

The examples in this chapter are written in C using a library called _libbpf_. You’ll learn more about this library in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]][[null|]][[null|]].[[null|]][[null|]]