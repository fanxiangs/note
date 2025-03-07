# Exercises

Here are some more ways to cause a verifier error. See if you can correlate the verifier log output to the errors you get:

1.  In [[#checking_memory_access|“Checking Memory Access”]], you saw the verifier rejecting access beyond the end of the global `message` array. In the example code there’s a section that accesses the local variable `data.message` in a similar way:
    
        if
    
    Try adjusting the code to make the same out-by-one mistake by replacing the `<` with `<=`, and you’ll see an error message about `invalid variable-offset read from stack R2`.
    
2.  Find the commented-out loops in _xdp\_hello_ in the example code. Try adding in the first loop that looks like this:
    
        for
    
    You should see in the verifier log a repeated series of lines that look something like this:
    
    42: (18) r1 = 0xffff800008e10009
    44: (b7) r2 = 11
    45: (b7) r3 = 8
    46: (85) call bpf\_trace\_printk#6
     R0=inv(id=0) R1\_w=map\_value(id=0,off=9,ks=4,vs=26,imm=0) R2\_w=inv11
     R3\_w=inv8 R6=pkt\_end(id=0,off=0,imm=0) R7=pkt(id=0,off=0,r=0,imm=0) 
     R10=fp0
    last\_idx 46 first\_idx 42
    regs=4 stack=0 before 45: (b7) r3 = 8
    regs=4 stack=0 before 44: (b7) r2 = 11
    
    From the log, work out which register is tracking the loop variable `i`.
    
3.  Now try adding in a loop that will fail, which looks like this:
    
        for
    
    You should see that the verifier tries to explore this loop to its conclusion, but it reaches the instruction complexity limit before it completes (because there is no upper bound on the global variable `c`).
    
4.  Write a program that attaches to a tracepoint. (You may have done this already for the exercises in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]].) Looking ahead to [[ch07.xhtml#tracepoints|“Tracepoints”]], you can see a structure definition for the context argument that starts with these fields:
    
        unsigned
    
    Create your own version of a structure that starts like this, and make the context argument in your program a pointer to this structure. In the program, try accessing any of these fields and see that the verifier fails with `invalid bpf_context access`.
    

[^1] For a long time the limit was 4,096 instructions, which imposed significant restrictions on the complexity of eBPF programs. This limit still applies to unprivileged users running BPF programs.

[^2] Helper functions are also defined in some other places in the source code, for example, [kernel/trace/bpf_trace.c](https://oreil.ly/cY8y9) and [net/core/filter.c](https://oreil.ly/qww-b).

[^3] This release brought a number of significant optimizations and improvements to the BPF verifier, which are summarized nicely in the LWN article [“Bounded loops in BPF for the 5.3 kernel”](https://oreil.ly/50BoD).