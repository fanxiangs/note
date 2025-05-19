# The JIT-Compiled Machine Code

[[null|]][[null|]][[null|]]The translated bytecode is pretty low level, but it’s not quite machine code yet. eBPF uses a JIT compiler to convert eBPF bytecode to machine code that runs natively on the target CPU. The `bytes_jited` field shows that after this conversation the program is 108 bytes long.

##### Note

For higher performance, eBPF programs are generally JIT-compiled. The alternative is to interpret the eBPF bytecode at runtime. The eBPF instruction set and registers were designed to map fairly closely to native machine instructions to make this interpretation straightforward and therefore relatively fast, but compiled programs will be faster, and most architectures now support JIT.[^5]

[[null|]]The `bpftool` utility can generate a dump of this JITed code in assembly language. Don’t worry if you’re not familiar with assembly language and this looks entirely incomprehensible! I have included it only to illustrate all the transformations the eBPF code goes through from source code to the executable machine instructions. Here is the command and its output:

$ bpftool prog dump jited name hello 
int hello(struct xdp\_md \* ctx):
bpf\_prog\_d35b94b4c0c10efb\_hello:
; bpf\_printk("Hello World %d", counter);
   0:   hint    #34
   4:   stp     x29, x30, \[sp, #-16\]!
   8:   mov     x29, sp
   c:   stp     x19, x20, \[sp, #-16\]!
  10:   stp     x21, x22, \[sp, #-16\]!
  14:   stp     x25, x26, \[sp, #-16\]!
  18:   mov     x25, sp
  1c:   mov     x26, #0
  20:   hint    #36
  24:   sub     sp, sp, #0
  28:   mov     x19, #-140733193388033
  2c:   movk    x19, #2190, lsl #16
  30:   movk    x19, #49152
  34:   mov     x10, #0
  38:   ldr     w2, \[x19, x10\]
  3c:   mov     x0, #-205419695833089
  40:   movk    x0, #709, lsl #16
  44:   movk    x0, #5904
  48:   mov     x1, #15
  4c:   mov     x10, #-6992
  50:   movk    x10, #29844, lsl #16
  54:   movk    x10, #56832, lsl #32
  58:   blr     x10
  5c:   add     x7, x0, #0
; counter++; 
  60:   mov     x10, #0
  64:   ldr     w0, \[x19, x10\]
  68:   add     x0, x0, #1
  6c:   mov     x10, #0
  70:   str     w0, \[x19, x10\]
; return XDP\_PASS;
  74:   mov     x7, #2
  78:   mov     sp, sp
  7c:   ldp     x25, x26, \[sp\], #16
  80:   ldp     x21, x22, \[sp\], #16
  84:   ldp     x19, x20, \[sp\], #16
  88:   ldp     x29, x30, \[sp\], #16
  8c:   add     x0, x7, #0
  90:   ret

##### Note

Some packaged distributions of `bpftool` don’t yet include support for dumping the JITed output, and if that’s the case, you’ll see “Error: No libbfd support.” You can build `bpftool` for yourself by following the instructions at [https://github.com/libbpf/bpftool](https://github.com/libbpf/bpftool).

You’ve seen that the “Hello World” program has been loaded into the kernel, but at this point it’s not yet associated with an event, so nothing will trigger it to run. It needs to be attached to an event.[[null|]][[null|]][[null|]]