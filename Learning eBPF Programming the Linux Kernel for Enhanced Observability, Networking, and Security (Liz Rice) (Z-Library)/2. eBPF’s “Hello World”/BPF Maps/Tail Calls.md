# Tail Calls
[[null|]][[null|]][[null|]]As described at [ebpf.io](https://oreil.ly/Loyuz), “tail calls can call and execute another eBPF program and replace the execution context, similar to how the `execve()` system call operates for regular processes.” In other words, execution doesn’t return to the caller after a tail call completes.

> [!Note]
[Tail calls](https://oreil.ly/cOA1r) are by no means exclusive to eBPF programming. The general motivation behind tail calls is to avoid adding frames to the stack over and over again as a function is called recursively, which can eventually lead to stack overflow errors. If you can arrange your code to call a recursive function as the last thing it does, the stack frame associated with the calling function isn’t really doing anything useful. Tail calls allow for calling a series of functions without growing the stack. This is particularly useful in eBPF where the [stack is limited to 512 bytes](https://oreil.ly/SZmkd).

Tail calls are made using the `bpf_tail_call()` helper function, which has the following signature:

```c
long bpf_tail_call(void *ctx, struct bpf_map *prog_array_map, u32 index)

```
The three arguments to this function have the following meanings:

*   `ctx` allows passing the context from the calling eBPF program to the callee.
*   `prog_array_map` is an eBPF map of type `BPF_MAP_TYPE_PROG_ARRAY`, which holds a set of file descriptors that identify eBPF programs.
*   `index` indicates which of that set of eBPF programs should be invoked.

This helper is somewhat unusual in that if it succeeds, it never returns. The currently running eBPF program is replaced on the stack by the program being called. The helper could fail, for example, if the indicated program doesn’t exist in the map, in which case the calling program carries on executing.

User space code has to load all the eBPF programs into the kernel (as usual), and it also sets up the program array map.

Let’s look at a simple example written in Python using BCC; you’ll find the code in the [GitHub repo](http://github.com/lizrice/learning-ebpf) as _chapter2/hello-tail.py_. The main eBPF program is attached to a tracepoint at the common entry point for all syscalls. This program uses tail calls to trace out specific messages for certain syscall opcodes. If there isn’t a tail call for a given opcode, the program traces out a generic message.

If you’re using the BCC framework, to make a [tail call](https://oreil.ly/rT9e1) you can use a line of the slightly simpler form:

```C
prog_array_map.call(ctx, index)
```

Before passing the code to the compilation step, BCC will rewrite the preceding line to this:

```C
bpf_tail_call(ctx, prog_array_map, index)
```

Here is the source code for the eBPF program and its tail calls:
```C
BPF_PROG_ARRAY(syscall, 300);                                   

int hello(struct bpf_raw_tracepoint_args *ctx) {                
   int opcode = ctx->args[1];                                   
   syscall.call(ctx, opcode);                                   
   bpf_trace_printk("Another syscall: %d", opcode);             
   return 0;
}

int hello_execve(void *ctx) {                                   
   bpf_trace_printk("Executing a program");
   return 0;
}

int hello_timer(struct bpf_raw_tracepoint_args *ctx) {          
   if (ctx->args[1] == 222) {
       bpf_trace_printk("Creating a timer");
   } else if (ctx->args[1] == 226) {
       bpf_trace_printk("Deleting a timer");
   } else {
       bpf_trace_printk("Some other timer operation");
   }
   return 0;
}

int ignore_opcode(void *ctx) {                                  
   return 0;
}
```

1. BCC provides a `BPF_PROG_ARRAY` macro for easily defining maps of type `BPF_MAP_TYPE_PROG_ARRAY`. I have called the map `syscall` and allowed for 300 entries,[^9] which is going to be sufficient for this example.
2. In the user space code that you’ll see shortly, I’m going to attach this eBPF program to the `sys_enter` raw tracepoint, which gets hit whenever any syscall is made. The context passed to an eBPF program attached to a raw tracepoint takes the form of this `bpf_raw_tracepoint_args` structure.
3. In the case of `sys_enter`, the raw tracepoint arguments include the opcode identifying which syscall is being made.
4. Here we make a tail call to the entry in the program array whose key matches the opcode. This line of code will be rewritten by BCC to a call to the `bpf_tail_call()` helper function before it passes the source code to the compiler.
5. If the tail call succeeds, this line tracing out the opcode value will never be hit. I’ve used this to provide a default line of trace for opcodes for which there isn’t a program entry in the map.
6. `hello_exec()` is a program that will be loaded into the syscall program array map, to be executed as a tail call when the opcode indicates it’s an `execve()` syscall. It’s just going to generate a line of trace to tell the user a new program is being executed.
7. `hello_timer()` is another program that will be loaded into the syscall program array. In this case it’s going to be referred to by more than one entry in the program array.
8. `ignore_opcode()` is a tail call program that does nothing. I’ll use this for syscalls where I don’t want any trace to be generated at all.

Now let’s look at the user space code that loads and manages this set of eBPF programs:

```C
b = BPF(text=program)                                              
b.attach_raw_tracepoint(tp="sys_enter", fn_name="hello")           

ignore_fn = b.load_func("ignore_opcode", BPF.RAW_TRACEPOINT)       
exec_fn = b.load_func("hello_exec", BPF.RAW_TRACEPOINT)
timer_fn = b.load_func("hello_timer", BPF.RAW_TRACEPOINT)

prog_array = b.get_table("syscall")                                
prog_array[ct.c_int(59)] = ct.c_int(exec_fn.fd)
prog_array[ct.c_int(222)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(223)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(224)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(225)] = ct.c_int(timer_fn.fd)
prog_array[ct.c_int(226)] = ct.c_int(timer_fn.fd)

# Ignore some syscalls that come up a lot                          
prog_array[ct.c_int(21)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(22)] = ct.c_int(ignore_fn.fd)
prog_array[ct.c_int(25)] = ct.c_int(ignore_fn.fd)
...

b.trace_print()
```
1. Instead of attaching to a kprobe, as you saw earlier, this time the user space code attaches the main eBPF program to the `sys_enter` tracepoint.
2. These calls to `b.load_func()` return a file descriptor for each tail call program. Notice that tail calls need to have the same program type as their parent—`BPF.RAW_TRACEPOINT` in this case. Also, it bears pointing out that each tail call program is an eBPF program in its own right.
3. The user space code creates entries in the `syscall` map. The map doesn’t have to be fully populated for every possible opcode; if there is no entry for a particular opcode, it simply means no tail call will be executed. Also, it’s perfectly fine to have multiple entries that point to the same eBPF program. In this case, I want the `hello_timer()` tail call to be executed for any of a set of timer-related syscalls.
4. Some syscalls get run so frequently by the system that a line of trace for each of them clutters up the trace output to the point of unreadability. I’ve used the `ignore_opcode()` tail call for several syscalls.
5. Print the trace output to the screen, until the user terminates the program.

Running this program generates trace output for every syscall that runs on the (virtual) machine, unless the opcode has an entry that links it to the `ignore_opcode()` tail call. Here’s some example output from running `ls` in another terminal (some details have been omitted for readability):

./hello-tail.py 
b'   hello-tail.py-2767    ... Another syscall: 62'
b'   hello-tail.py-2767    ... Another syscall: 62'
...
b'            bash-2626    ... Executing a program'
b'            bash-2626    ... Another syscall: 220'
...
b'           <...>-2774    ... Creating a timer'
b'           <...>-2774    ... Another syscall: 48'
b'           <...>-2774    ... Deleting a timer'
...
b'              ls-2774    ... Another syscall: 61'
b'              ls-2774    ... Another syscall: 61'
...

The particular syscalls being executed are beside the point, but you can see that the different tail calls are getting called and are generating trace messages. You can also see the default message `Another syscall` for opcodes that don’t have an entry in the tail call program map.

> [!Note]
Check out Paul Chaignon’s blog post about the [cost of BPF tail calls](https://oreil.ly/jTxcb) on various different kernel versions.

Tail calls have been supported in eBPF since kernel version 4.2, but for a long time they were incompatible with making BPF to BPF function calls. This restriction was lifted in kernel 5.10.[^10]

The fact that you can chain up to 33 tail calls together, combined with the instruction complexity limit per eBPF program of 1 million instructions, means that today’s eBPF programmers have a lot of leeway to write very complex code to run entirely in the kernel[[null|]][[null|]][[null|]].