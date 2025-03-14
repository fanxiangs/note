# Checking Memory Access

[[null|]][[null|]]The verifier performs a number of checks to make sure BPF programs only access memory they are supposed to have access to.

[[null|]]For example, when processing a network packet, an XDP program is only permitted to access the memory locations that make up that network packet. Most XDP programs start with something very similar to the following:

    SEC

The `xdp_md` structure passed as the context to the program describes the network packet that has been received. The `ctx->data` field within that structure is the location in memory where the packet starts, and `ctx->data_end` is the last location in the packet. The verifier will ensure that the program doesn’t exceed these bounds.

For example, the following program in _hello\_verifier.bpf.c_ is valid:

    SEC

The variables `data` and `data_end` are very similar, but the verifier is smart enough to recognize that `data_end` relates to the end of a packet. Your program is required to check that any values read from the packet aren’t from beyond that location, and it won’t let you “cheat” by modifying the `data_end` value. Try adding the following line just before the `bpf_printk()` call:

    data_end

The verifier will complain, like this:

; data\_end++;
1: (07) r3 += 1
R3 pointer arithmetic on pkt\_end prohibited

In another example, when accessing an array you need to make sure there’s no possibility of accessing an index that is beyond the bounds of that array. In the example code there is a section that reads a character out of the `message` array, like this:

    if

This is fine because of the explicit check to ensure that the counter variable `c` is no bigger than the size of the message array. Making a simple “off by one” error like the following renders it invalid:

    if

The verifier will fail this with an error message similar to this:

invalid access to map value, value\_size=16 off=16 size=1
R2 max value is outside of the allowed memory range

It’s fairly clear from this message that there is an invalid access to a map value because Register 2 might hold a value that’s too large for indexing the map. If you were debugging this error, you’d want to dig into the log to see what line in the source code was responsible. The log ends like this just before emitting the error message (I have removed some of the state information for clarity):

; if (c <= sizeof(message)) {
30: (25) if r1 > 0xc goto pc+10                                [[#list_id_6_4|]]
 R0\_w=map\_value\_or\_null(id=2,off=0,ks=4,vs=12,imm=0) R1\_w=inv(id=0,
 umax\_value=12,var\_off=(0x0; 0xf)) R6=ctx(id=0,off=0,imm=0) ...
; char a = message\[c\];
31: (18) r2 = 0xffff800008e00004                               [[#list_id_6_5|]]
33: (0f) r2 += r1                                               
last\_idx 33 first\_idx 19
regs=2 stack=0 before 31: (18) r2 = 0xffff800008e00004
regs=2 stack=0 before 30: (25) if r1 > 0xc goto pc+10
regs=2 stack=0 before 29: (61) r1 = \*(u32 \*)(r8 +0)
34: (71) r3 = \*(u8 \*)(r2 +0)                                   [[#list_id_6_6|]]
 R0\_w=map\_value\_or\_null(id=2,off=0,ks=4,vs=12,imm=0) R1\_w=invP(id=0,
 umax\_value=12,var\_off=(0x0; 0xf)) R2\_w=map\_value(id=0,off=4,ks=4,vs=16,
 umax\_value=12,var\_off=(0x0; 0xf),s32\_max\_value=15,u32\_max\_value=15)
 R6=ctx(id=0,off=0,imm=0) ...

[[#code_id_6_6|]]

Working backward from the error, the last register state information shows that Register 2 could have a maximum value of `12`.

[[#code_id_6_5|]]

At instruction 31, Register 2 is set to an address in memory and then is incremented by the value of Register 1. The output shows that this corresponds to the line of code accessing `message[c]`, so it stands to reason that Register 2 is set to point to the message array and then to be incremented by the value of `c`, which is held in the Register 1 register.

[[#code_id_6_4|]]

Working further back to find the value of Register 1, the log shows that it has a maximum value of `12` (which is hex 0x0c). However, `message` is defined as a 12-byte character array, so only indexes 0 through 11 are within its bounds. From this, you can see that the error springs from the source code testing for `c <= sizeof(message)`.

At step 2, I have inferred the relationship between some registers and the source code variables they represent, from the lines of source code the verifier has helpfully included in the log. You could work back through the verifier log to check that this is true, and indeed you might have to if the code was compiled without debug information. Given the debug information is present, it makes sense to use it.

The `message` array is declared as a global variable, and you might recall from [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]] that global variables are implemented using maps. This explains why the error message talks about “invalid access to a map value.”[[null|]][[null|]]