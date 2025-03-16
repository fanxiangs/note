# Loops

[[null|]][[null|]]To guarantee completion, until kernel version 5.3 there was a restriction on loops.[^3] Looping through the same instructions requires a jump backward to earlier instructions, and it used to be the case that the verifier would not permit this. eBPF programmers worked around this by using the `#pragma unroll` compiler directive to tell the compiler to write out a set of identical (or very similar) bytecode instructions for each time around the loop. This saved the programmer typing in the same lines many times, but you would see repeated instructions in the emitted bytecode.

From version 5.3 onward the verifier follows branches backward as well as forward as part of its process of checking all the possible execution paths. This means it can accept some loops, provided the execution path remains within the limit of one million instructions.

You can see an example of a loop in the example _xdp\_hello_ program. A version of the loop that passes verification looks like this:

    for

The (successful) verifier log will show that it has followed the execution path around this loop 10 times. In doing so, it doesn’t hit the complexity limit of one million instructions. In the exercises for this chapter, there’s another version of this loop that will hit that limit and will fail verification.

In version 5.17 a new helper function, `bpf_loop()`, was introduced that makes it much easier for the verifier not only to accept loops but also to do it in a much more efficient way. This helper takes the maximum number of iterations as its first argument, and it is also passed a function that is called for each iteration. The verifier only has to validate the BPF instructions in that function once, however many times it might be called. That function can return a nonzero value to indicate that there is no need to call it again, which is used to terminate a loop early once the desired result is achieved.

There’s also a helper function [bpf_for_each_map_elem()](https://oreil.ly/Yg_oQ) that calls a provided callback function for each item in a map.