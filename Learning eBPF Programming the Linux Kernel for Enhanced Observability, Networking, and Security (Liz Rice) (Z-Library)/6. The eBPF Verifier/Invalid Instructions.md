# Invalid Instructions

[[null|]][[null|]]As you know from the discussion of the eBPF (virtual) machine in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]], eBPF programs consist of a set of bytecode instructions. The verifier checks that the instructions in a program are valid bytecode instructions—for example, using only known opcodes.

It would be considered a bug in the compiler if it emitted invalid bytecode, so you’re not likely to see this kind of verifier error unless you choose (for some reason best known to yourself) to write eBPF bytecode by hand. However, there have been some instructions added more recently, such as the atomic operations. If your compiled bytecode uses these instructions, they would fail verification on an older kernel.