# Checking the Return Code

[[null|]][[null|]]The return code from an eBPF program is stored in Register 0 (`R0`). If the program leaves `R0` uninitialized, the verifier will fail, like this:

R0 !read\_ok

You can try this by commenting out all the code in a function; for example, modify the `xdp_hello` example to be like this:

    SEC

This will fail the verifier. However, if you put the line with the helper function `bpf_printf()` back in, the verifier won’t complain, even though there’s no explicit return value set by the source code!

This is because Register 0 is also used to hold the return code from a helper function. After returning from a helper function in an eBPF program, Register 0 is no longer uninitialized.