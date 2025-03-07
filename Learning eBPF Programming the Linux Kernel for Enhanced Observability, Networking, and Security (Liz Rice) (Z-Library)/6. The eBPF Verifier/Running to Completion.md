# Running to Completion

[[null|]][[null|]]The verifier ensures that the eBPF program will run to completion; otherwise, there is a risk that it might consume resources indefinitely. It does this by having a limit on the total number of instructions that it will process, which, as I mentioned earlier, is set at one million instructions at the time of this writing. That limit is [hard-coded into the kernel](https://oreil.ly/IucYm); it’s not a configurable option. If the verifier hasn’t reached the end of the BPF program before it has processed this many instructions, it rejects the program.

One easy way to create a program that never completes is to write a loop that never ends. Let’s see how loops can be created in eBPF programs.