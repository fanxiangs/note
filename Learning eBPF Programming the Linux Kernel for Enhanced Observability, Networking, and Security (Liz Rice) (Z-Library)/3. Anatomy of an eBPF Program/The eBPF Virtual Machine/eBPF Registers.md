# eBPF Registers

[[null|]][[null|]]The eBPF virtual machine uses 10 general-purpose registers, numbered 0 to 9. Additionally, Register 10 is used as a stack frame pointer (and can only be read, but not written). As a BPF program is executed, values get stored in these registers to keep track of state.

It’s important to understand that these eBPF registers in the eBPF virtual machine are implemented in software. You can see them enumerated from `BPF_REG_0` to `BPF_REG_10` in the [include/uapi/linux/bpf.h header file](https://oreil.ly/_ZhU2) of the Linux kernel’s source code.

The context argument to an eBPF program is loaded into Register 1 before its execution begins. The return value from the function is stored in Register 0.

Before calling a function from eBPF code, the arguments to that function are placed in Register 1 through Register 5 (not all the registers are used if there are fewer than five arguments).