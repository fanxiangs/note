# The Verifier Log

[[null|]][[null|]]When the verification of a program fails, the verifier generates a log showing how it reached the conclusion that the program is invalid. If you’re using `bpftool prog load`, the verifier log gets output to stderr. When you’re writing a program with _libbpf_, you can use the function `libbpf_set_print()` to set a handler that will display (or do something else useful with) any errors. (You’ll see an example of this in the _hello-verifier.c_ source code for this chapter.)

###### Note

If you really want to dig into what the verifier is doing, you can get it to generate the log on success as well as on failure. There is a basic example of this in the _hello-verifier.c_ file too. It involves passing a buffer that will hold verifier log contents into the _libbpf_ call that loads the program into the kernel and then writing the contents of that log to screen.

The verifier log includes a summary of how much work the verifier did, which looks something like this:

processed 61 insns (limit 1000000) max\_states\_per\_insn 0 total\_states 4
peak\_states 4 mark\_read 3

In this example, the verifier processed 61 instructions, including potentially processing the same instruction multiple times by arriving at it through different paths. Note that the complexity limit of one million is an upper bound on the number of instructions in a program; in practice, if there are branches in the code, the verifier will process some instructions more than once.

The total number of states stored was four, which for this simple program matches the peak number of stored states. If some of the states had been pruned, the peak number might be lower than the total.

The log output includes the BPF instructions the verifier has analyzed, along with the corresponding C source code lines (if the object file was built with the `-g` flag to include debug information) and summaries of verifier state information. Here is an example extract of the verifier log relating to the first few lines of the program in _hello-verifier.bpf.c_:

0: (bf) r6 = r1
; data.counter = c;                                              [[#list_id_6_1|]]
1: (18) r1 = 0xffff800008178000
3: (61) r2 = \*(u32 \*)(r1 +0)
 R1\_w=map\_value(id=0,off=0,ks=4,vs=16,imm=0) R6\_w=ctx(id=0,off=0,imm=0) 
 R10=fp0                                                         [[#list_id_6_2|]]
; c++; 
4: (bf) r3 = r2
5: (07) r3 += 1
6: (63) \*(u32 \*)(r1 +0) = r3
 R1\_w=map\_value(id=0,off=0,ks=4,vs=16,imm=0) R2\_w=inv(id=1,umax\_value=4294967295,
 var\_off=(0x0; 0xffffffff)) R3\_w=inv(id=0,umin\_value=1,umax\_value=4294967296,
 var\_off=(0x0; 0x1ffffffff)) R6\_w=ctx(id=0,off=0,imm=0) R10=fp0  [[#list_id_6_3|]]

[[#code_id_6_1|]]

The log includes source code lines to make it easier to understand how the output relates to the source. This source code is available because the `-g` flag was used to build in debug information during the compilation step.

[[#code_id_6_2|]]

Here’s an example of some register state information being output in the log. It tells us that at this stage Register 1 contains a map value, Register 6 holds the context, and Register 10 is the frame (or stack) pointer where local variables are held.

[[#code_id_6_3|]]

This is another example of register state information. Here you can see not only the types of values that are held in each (initialized) register, but also the range of possible values for Register 2 and Register 3.

Let’s dig further into the details of this. I said that Register 6 holds the context, and the verifier log indicates this with `R6_w=ctx(id=0,off=0,imm=0)`. This was set up in the very first line of the bytecode, where Register 1 was copied to Register 6. When an eBPF program is called, Register 1 always holds the context argument passed to the program. Why copy it to Register 6? Well, when a BPF helper function is called, the arguments to that call are passed in Registers 1 through 5. Helper functions don’t modify the contents of Registers 6 through 9, so saving the context off into Register 6 means the code can call a helper function without losing access to the context.

Register 0 is used for the return value from a helper function and also for the return value from an eBPF program. Register 10 always holds a pointer to the eBPF stack frame (and the eBPF program can’t modify it).

Let’s look at the register state information for Registers 2 and 3 after instruction 6:

R2\_w=inv(id=1,umax\_value=4294967295,var\_off=(0x0; 0xffffffff))
R3\_w=inv(id=0,umin\_value=1,umax\_value=4294967296,var\_off=(0x0; 0x1ffffffff))

Register 2 doesn’t have a minimum value, and the `umax_value` shown here in decimal corresponds to 0xFFFFFFFF, which is the largest value that can be held in an 8-byte register. In other words, at this point the register could hold any of its possible values.

In instruction 4, the contents of Register 2 are copied into Register 3, and then instruction 5 adds one to that value. Therefore, Register 3 could have any value that’s 1 or greater. You can see this in the state information for Register 3, which has `umin_value` set to `1`, and a `umax_value` of `0xFFFFFFFF`.

The verifier uses the information about not just the states of each register, but also the range of values each can contain, to determine the possible paths through the program. This is also used for the state pruning that I mentioned before: if the verifier has been in the same position in the code, with the same types and possible ranges of values for each register, there’s no need to evaluate this path further. What’s more, if the current state is a subset of a state that was seen earlier, it can also be pruned.[[null|]][[null|]]