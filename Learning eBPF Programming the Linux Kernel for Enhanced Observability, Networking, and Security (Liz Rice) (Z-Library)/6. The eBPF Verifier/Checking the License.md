# Checking the License

[[null|]][[null|]][[null|]]The verifier also checks that if you are using a BPF helper function that’s licensed under GPL, your program also has a GPL-compatible license. The last line in the [[#the_ebpf_verifier|Chapter 6]] example code _hello-verifier.bpf.c_ defines a “license” section that holds the string `Dual BSD/GPL`. If you remove this line, the output from the verifier will end like this:

...
37: (85) call bpf\_probe\_read\_kernel#113
cannot call GPL-restricted function from non-GPL compatible program

That’s because the `gpl_only` field is set to `true` for the `bpf_probe_read_kernel()` helper function. There are other helper functions called earlier in this eBPF program, but they are not GPL licensed, so the verifier doesn’t object to their use.

The BCC project maintains a [list of helper functions](https://oreil.ly/mCpvB), indicating whether they are GPL licensed or not. If you’re interested in more details on how helper functions are implemented, there’s a section on this in the [BPF and XDP reference guide](https://oreil.ly/kVd6j).