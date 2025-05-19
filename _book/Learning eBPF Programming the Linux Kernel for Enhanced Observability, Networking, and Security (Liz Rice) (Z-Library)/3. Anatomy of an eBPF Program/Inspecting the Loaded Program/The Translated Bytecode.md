# The Translated Bytecode

[[null|]][[null|]]The `bytes_xlated` field tells us how many bytes of “translated” eBPF code there are. This is the eBPF bytecode after it has passed through the verifier (and possibly been modified by the kernel for reasons I’ll discuss later in this book).

[[null|]]Let’s use `bpftool` to show this translated version of our “Hello World” code:

$ bpftool prog dump xlated name hello 
int hello(struct xdp\_md \* ctx):
; bpf\_printk("Hello World %d", counter);
   0: (18) r6 = map\[id:165\]\[0\]+0
   2: (61) r3 = \*(u32 \*)(r6 +0)
   3: (18) r1 = map\[id:166\]\[0\]+0
   5: (b7) r2 = 15
   6: (85) call bpf\_trace\_printk#-78032
; counter++; 
   7: (61) r1 = \*(u32 \*)(r6 +0)
   8: (07) r1 += 1
   9: (63) \*(u32 \*)(r6 +0) = r1
; return XDP\_PASS;
  10: (b7) r0 = 2
  11: (95) exit

This looks very similar to the disassembled code you saw earlier in the output from `llvm-objdump`. The offset addresses are the same, and the instructions look similar—for example, we can see the instruction at offset `5` is `r2=15`.