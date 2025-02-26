# Optimization

[[null|]][[null|]]The `-O2` optimization flag (level 2 or higher) is required for Clang to produce BPF bytecode that will pass the verifier. One example of this being necessary is that, by default, Clang will output `callx` to call helper functions, but eBPF doesn’t support calling addresses from registers.