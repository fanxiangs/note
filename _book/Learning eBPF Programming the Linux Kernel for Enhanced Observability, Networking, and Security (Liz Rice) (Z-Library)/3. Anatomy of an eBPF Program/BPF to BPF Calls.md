# BPF to BPF Calls

[[null|]][[null|]][[null|]]In the previous chapter you saw tail calls in action, and I mentioned that now there is also the ability to call functions from within an eBPF program. Let’s take a look at a simple example, which, like the tail call example, can be attached to the `sys_enter` tracepoint, except this time it will trace out the opcode for the syscall. You’ll find the code in _chapter3/hello-func.bpf.c_.

For illustrative purposes I have written a very simple function that extracts the syscall opcode from the tracepoint arguments:

    static

Given the choice, the compiler would probably inline this very simple function that I’m only going to call from one place. Since that would defeat the point of this example, I have added `__attribute((noinline))` to force the compiler’s hand. In normal circumstances you should probably omit this and allow the compiler to optimize as it sees fit.

The eBPF function that calls this function looks like this:

    SEC

[[null|]]After compiling this to an eBPF object file, you can load it into the kernel and confirm that it is loaded with `bpftool`:

$ bpftool prog load hello-func.bpf.o /sys/fs/bpf/hello
$ bpftool prog list name hello 
893: raw\_tracepoint  name hello  tag 3d9eb0c23d4ab186  gpl
        loaded\_at 2023-01-05T18:57:31+0000  uid 0
        xlated 80B  jited 208B  memlock 4096B  map\_ids 204
        btf\_id 302

The interesting part of this exercise is inspecting the eBPF bytecode to see the `get_opcode()` function:

$ bpftool prog dump xlated name hello 
int hello(struct bpf\_raw\_tracepoint\_args \* ctx):
; int opcode = get\_opcode(ctx);                            [[#list_id_3_15|]]
   0: (85) call pc+7#bpf\_prog\_cbacc90865b1b9a5\_get\_opcode
; bpf\_printk("Syscall: %d", opcode);
   1: (18) r1 = map\[id:193\]\[0\]+0
   3: (b7) r2 = 12
   4: (bf) r3 = r0
   5: (85) call bpf\_trace\_printk#-73584
; return 0;
   6: (b7) r0 = 0
   7: (95) exit
int get\_opcode(struct bpf\_raw\_tracepoint\_args \* ctx):      [[#list_id_3_16|]]
; return ctx->args\[1\];
   8: (79) r0 = \*(u64 \*)(r1 +8)
; return ctx->args\[1\];
   9: (95) exit

[[#code_id_3_15|]]

Here you can see the `hello()` eBPF program making a call to `get_opcode()`. The eBPF instruction at offset `0` is `0x85`, which from the instruction set documentation corresponds to “Function call.” Instead of executing the next instruction, which would be at offset 1, execution will jump seven instructions ahead (`pc+7`), which means the instruction at offset `8`.

[[#code_id_3_16|]]

Here’s the bytecode for `get_opcode()`, and as you might hope, the first instruction is at offset `8`.

The function call instruction necessitates putting the current state on the eBPF virtual machine’s stack so that when the called function exits, execution can continue in the calling function. Since the stack size is limited to 512 bytes, BPF to BPF calls can’t be very deeply nested.

###### Note

For a lot more detail on tail calls and BPF to BPF calls, there’s an excellent post by Jakub Sitnicki on Cloudflare’s blog: [“Assembly within! BPF tail calls on x86 and ARM”](https://oreil.ly/6kOp3).[[null|]][[null|]][[null|]]