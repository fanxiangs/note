# Inspecting an eBPF Object File

[[null|]][[null|]]The file utility is commonly used to determine the contents of a file:

$ file hello.bpf.o
hello.bpf.o: ELF 64-bit LSB relocatable, eBPF, version 1 (SYSV), with debug\_info,
not stripped

This shows it’s an ELF (Executable and Linkable Format) file, containing eBPF code, for a 64-bit platform with LSB (least significant bit) architecture. It includes debug information if you used the `-g` flag at the compilation step.

[[null|]]You can inspect this object further with `llvm-objdump` to see the eBPF instructions:

$ llvm-objdump -S hello.bpf.o

Even if you’re not familiar with disassembly, the output from this command isn’t too hard to understand:

hello.bpf.o:    file format elf64-bpf               [[#list_id_3_9|]]

Disassembly of section xdp:                         [[#list_id_3_10|]]

0000000000000000 :                           [[#list_id_3_11|]]
;  bpf\_printk("Hello World %d", counter");          [[#list_id_3_12|]] 
    0:   18 06 00 00 00 00 00 00 00 00 00 00 00 00 00 00 r6 = 0 ll
    2:   61 63 00 00 00 00 00 00 r3 = \*(u32 \*)(r6 + 0)
    3:   18 01 00 00 00 00 00 00 00 00 00 00 00 00 00 00 r1 = 0 ll
    5:   b7 02 00 00 0f 00 00 00 r2 = 15
    6:   85 00 00 00 06 00 00 00 call 6
;  counter++;                                       [[#list_id_3_13|]]
    7:   61 61 00 00 00 00 00 00 r1 = \*(u32 \*)(r6 + 0)
    8:   07 01 00 00 01 00 00 00 r1 += 1
    9:   63 16 00 00 00 00 00 00 \*(u32 \*)(r6 + 0) = r1
;  return XDP\_PASS;                                 [[#list_id_3_14|]]
   10:   b7 00 00 00 02 00 00 00 r0 = 2
   11:   95 00 00 00 00 00 00 00 exit

[[#code_id_3_9|]]

The first line gives further confirmation that _hello.bpf.o_ is a 64-bit ELF file with eBPF code (there’s no particular rhyme or reason why some tools use the term _BPF_ and others _eBPF_; as I said earlier, these terms are now practically interchangeable).

[[#code_id_3_10|]]

Next comes the disassembly of the section labeled `xdp`, which matches the `SEC()` definition in the C source code.

[[#code_id_3_11|]]

This section is a function called `hello`.

[[#code_id_3_12|]]

There are five lines of eBPF bytecode instructions that correspond to the source line `bpf_printk("Hello World %d", counter");`.

[[#code_id_3_13|]]

Three lines of eBPF bytecode instructions increment the `counter` variable.

[[#code_id_3_14|]]

And another two lines of bytecode are generated from the source code `return XDP_PASS;`.

Unless you’re particularly keen to do so, there’s no real need to understand exactly how each line of bytecode relates to the source. The compiler takes care of generating the bytecode so that you don’t have to think about it! But let’s examine the output in a little more detail so you can get a feel for how this output relates to the eBPF instructions and registers you learned about earlier in this chapter.

To the left of each line of bytecode you can see the offset of that instruction from wherever `hello` is located in memory. As described earlier in this chapter, eBPF instructions are generally 8 bytes long, and since on a 64-bit platform each memory location can hold 8 bytes, the offset is usually incremented by one for each instruction. [[null|]]However, the first instruction in this program happens to be a wide instruction encoding that requires 16 bytes in order to set Register 6 to a 64-bit value of `0`. That places the instruction in the second line of output at offset `2`. After that there is another 16-byte instruction, setting Register 1 to a 64-bit value of `0`. And after that, the remaining instructions each fit in 8 bytes, so the offset increments by one in each line.

The first byte of each line is the opcode that tells the kernel what operation to perform, and on the right side of each instruction line is the human-readable interpretation of the instruction. [[null|]]At the time of this writing, the Iovisor project has the most complete [documentation](https://oreil.ly/nLbLp) of the eBPF opcodes, but the official [Linux kernel documentation](https://oreil.ly/yp-jW) is catching up, and the eBPF Foundation is working on [standard documentation](https://oreil.ly/7ZWzj) that is not tied to a specific operating system.

For example, let’s take the instruction at offset `5`, which looks like this:

    5:   b7 02 00 00 0f 00 00 00 r2 = 15

The opcode is `0xb7`, and the documentation tells us the pseudocode corresponding to this is `dst = imm`, which can be read as “Set the destination to the immediate value.” The destination is defined by the second byte, `0x02`, which means “Register 2.” The “immediate” (or literal) value here is `0x0f`, which is 15 in decimal. So we can understand that this instruction tells the kernel to “set Register 2 to value 15.” This corresponds to the output we see on the right side of the instruction: `r2 = 15`.

The instruction at offset `10` is similar:

   10:   b7 00 00 00 02 00 00 00 r0 = 2

This line also has opcode `0xb7`, and this time it’s setting the value of Register 0 to `2`. When an eBPF program finishes running, Register 0 holds the return code, and `XDP_PASS` has the value `2`. This matches the source code, which always returns `XDP_PASS`.

You now know that _hello.bpf.o_ contains an eBPF program in bytecode. The next step is to load it into the kernel.[[null|]][[null|]]