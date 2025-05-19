# The Verification Process

[[null|]]The verifier analyzes the program to assess all possible execution paths. It steps through the instructions in order, evaluating them rather than actually executing them. As it goes along it keeps track of the state of each register in a structure called `bpf_reg_state`. (The registers I’m referring to here are the registers from the eBPF virtual machine that you met in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]].) This structure includes a field called `bpf_reg_type`, which describes what type of value is held in that register. There are several possible types, including these:

*   `NOT_INIT`, indicating that the register has not yet been set to a value.
    
*   `SCALAR_VALUE`, indicating that the register has been set to a value that doesn’t represent a pointer.
    
*   Several `PTR_TO_*` types, indicating that the register holds a pointer to something. That something could be, for example:
    
    *   `PTR_TO_CTX`: The register holds a pointer to the context passed as the argument to a BPF program.
        
    *   `PTR_TO_PACKET`: The register points to a network packet (held in the kernel as `skb->data`).
        
    *   `PTR_TO_MAP_KEY` or `PTR_TO_MAP_VALUE`: I’m sure you can guess what these mean.
        

There are several other `PTR_TO_*` types, and you can find the full set enumerated in the [linux/bpf.h header file](https://oreil.ly/aWb50).

The `bpf_reg_state` structure also keeps track of the range of possible values the register might hold. This information is used by the verifier to determine when invalid actions are being attempted.

Each time the verifier comes to a branch, where a decision has to be made on whether to carry on in sequence or jump to a different instruction, the verifier pushes a copy of the current state of all the registers onto a stack and explores one of the possible paths. It continues evaluating the instructions until it reaches the return at the end of the program (or reaches the limit on the number of instructions it will process, which is currently one million instructions[^1]), at which point it pops a branch off the stack to evaluate next. If it finds an instruction that could result in an invalid operation, it fails verification.

[[null|]]Verifying every single possibility could get computationally expensive, so in practice there are optimizations called _state pruning_ that avoid reevaluating paths through the program that are essentially equivalent. As it works through the program, the verifier records the state of all the registers at certain instructions within the program. If it later arrives at the same instruction with registers in a matching state, there is no need to continue to verify the rest of that path, as it’s already known to be valid.

[Lots of work has gone into optimizing the verifier](https://oreil.ly/pQDES) and its pruning process. The verifier used to store pruning state before and after each jump instruction, but analysis showed that this results in storing state on average every four instructions or so, and the vast majority of these pruning states would never get matched. It turned out that it’s more efficient to store pruning state every 10 instructions, regardless of branching.

###### Note

You can read more details on how verification works in the [kernel documentation](https://oreil.ly/atNda).[[null|]]