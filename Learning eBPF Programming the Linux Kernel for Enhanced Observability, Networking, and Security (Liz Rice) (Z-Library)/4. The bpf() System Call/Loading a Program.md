# Loading a Program

[[null|]][[null|]]So far you have seen the example program using syscalls to load BTF data into the kernel and create some eBPF maps. The next thing it does is load the eBPF program being loaded into the kernel with the following `bpf()` syscall:

bpf(BPF\_PROG\_LOAD, {prog\_type=BPF\_PROG\_TYPE\_KPROBE, insn\_cnt=44,
insns=0xffffa836abe8, license="GPL", ... prog\_name="hello", ... 
expected\_attach\_type=BPF\_CGROUP\_INET\_INGRESS, prog\_btf\_fd=3,...}, 128) = 6

Quite a few of the fields here are interesting:

*   The `prog_type` field describes the program type, which here indicates that it’s intended to be attached to a kprobe. You’ll learn more about program types in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].
    
*   The `insn_cnt` field means “instruction count.” This is the number of bytecode instructions in the program.
    
*   The bytecode instructions that make up this eBPF program are held in memory at the address specified in the `insns` field.
    
*   This program was specified as GPL licensed so that it can use GPL-licensed BPF helper functions.
    
*   The program name is `hello`.
    
*   The `expected_attach_type` of `BPF_CGROUP_INET_INGRESS` might seem surprising, because that sounds like something to do with ingress network traffic, but you know this eBPF program is going to be attached to a kprobe. In fact, the `expected_attach_type` field is only used for some program types, and `BPF_PROG_TYPE_KPROBE` isn’t one of them. `BPF_CGROUP_INET_INGRESS` just happens to be the first in the list of BPF attachment types,[^3] so it has the value `0`.
    
*   The `prog_btf_fd` field tells the kernel which blob of previously loaded BTF data to use with this program. The value `3` here corresponds to the file descriptor you saw returned from the `BPF_BTF_LOAD` syscall (and it’s the same blob of BTF data used for the `config` map).
    

If the program had failed verification (which I’ll discuss in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]]), this syscall would have returned a negative value, but here you can see it returned the file descriptor 6. To recap, at this point the file descriptors have the meanings shown in [[#file_descriptors_when_running_hello_buf|Table 4-1]].

Table 4-1. File descriptors when running hello-buffer-config.py after loading the program

File descriptor

Represents

`3`

BTF data

`4`

`output` perf buffer map

`5`

`config` hash table map

`6`

`hello` eBPF program