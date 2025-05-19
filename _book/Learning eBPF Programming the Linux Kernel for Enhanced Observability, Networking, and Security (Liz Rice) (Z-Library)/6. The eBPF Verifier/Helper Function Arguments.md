# Helper Function Arguments

[[null|]]If you look, for example, in [kernel/bpf/helpers.c](https://oreil.ly/tjjVR),[^2] you’ll find that each helper function has a `bpf_func_proto` structure similar to this example for the helper function `bpf_map_lookup_elem()`:

    const

This structure defines the constraints for arguments to and return values from the helper function. Because the verifier is keeping track of the type of value held in each register, it can spot if you try to pass the wrong kind of argument to a helper function. For example, try changing the argument to the call to `bpf_map_lookup_elem()` in the _hello_ program, like this:

    p

Instead of passing `&my_config`, which is a pointer to a map, this now passes `&data`, which is a pointer to a local variable structure. This is valid from the compiler’s point of view, so you can build the BPF object file _hello-verifier.bpf.o_, but when you try to load the program into the kernel, you’ll see an error like this in the verifier log:

27: (85) call bpf\_map\_lookup\_elem#1
R1 type=fp expected=map\_ptr

[[null|]]Here, `fp` stands for _frame pointer_, and it’s the area of memory on the stack where local variables are stored. Register 1 was loaded with the address of the local variable called `data`, but the function expects a pointer to a map (as indicated by the `arg1_type` field in the `bpf_func_proto` structure shown earlier). By tracking the types of value stored in each register, the verifier was able to spot this discrepancy.