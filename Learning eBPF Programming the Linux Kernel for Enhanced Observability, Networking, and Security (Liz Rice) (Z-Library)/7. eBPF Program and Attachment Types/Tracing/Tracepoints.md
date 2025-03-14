# Tracepoints

[[null|]][[null|]][Tracepoints](https://oreil.ly/yXk_L) are marked locations in the kernel code (we’ll come to user space tracepoints later in this chapter). They’re not by any means exclusive to eBPF and have long been used to generate kernel trace output and by tools like [SystemTap](https://oreil.ly/bLmQL). Unlike attaching to arbitrary instructions using kprobes, tracepoints are stable between kernel releases (although an older kernel might not have the full set of tracepoints that have been added into a newer one).

You can see the available set of tracing subsystems on your kernel by looking at _/sys/kernel/tracing/available\_events_, as follows:

$ cat /sys/kernel/tracing/available\_events 
tls:tls\_device\_offload\_set
tls:tls\_device\_decrypted
...
syscalls:sys\_exit\_execveat
syscalls:sys\_enter\_execveat
syscalls:sys\_exit\_execve
syscalls:sys\_enter\_execve
...

My 5.15 version of the kernel has more than 1,400 tracepoints defined in this list. The section definition for a tracepoint eBPF program should match one of these items so that _libbpf_ can automatically attach it to the tracepoint. The definition is in the form `SEC("tp/tracing subsystem/tracepoint name")`.

You’ll find an example in the _chapter7/hello.bpf.c_files that matches the `syscalls:sys_enter_execve` tracepoint that gets hit when the kernel starts processing an `execve()` call. The section definition tells _libbpf_ that this is a tracepoint program, and where it should be attached, like this:

    SEC

What about the context parameter to a tracepoint? As I’ll come to shortly, BTF can help us here, but first let’s consider what is needed when BTF isn’t available. Each tracepoint has a format describing the fields that get traced out from it. As an example, here’s the format for the tracepoint at the entry to the `execve()` syscall:

$ cat /sys/kernel/tracing/events/syscalls/sys\_enter\_execve/format
name: sys\_enter\_execve
ID: 622
format:
  field:unsigned short common\_type;         offset:0;  size:2; signed:0;
  field:unsigned char common\_flags;         offset:2;  size:1; signed:0;
  field:unsigned char common\_preempt\_count; offset:3;  size:1; signed:0;
  field:int common\_pid;                     offset:4;  size:4; signed:1;

  field:int \_\_syscall\_nr;                   offset:8;  size:4; signed:1;
  field:const char \* filename;              offset:16; size:8; signed:0;
  field:const char \*const \* argv;           offset:24; size:8; signed:0;
  field:const char \*const \* envp;           offset:32; size:8; signed:0;

print fmt: "filename: 0x%08lx, argv: 0x%08lx, envp: 0x%08lx", 
((unsigned long)(REC->filename)), ((unsigned long)(REC->argv)), 
((unsigned long)(REC->envp))

I used this information to define a matching structure called `m⁠y⁠_⁠s⁠y⁠s⁠c⁠a⁠l⁠l⁠s⁠_⁠e⁠n⁠t⁠e⁠r⁠_​e⁠x⁠e⁠c⁠v⁠e` in _chapter7/hello.bpf.c_:

    struct

eBPF programs aren’t allowed to access the first four of these fields. If you try to access them, the program will fail verification with an `invalid bpf_context access` error.

My example eBPF program that attaches to this tracepoint can use a pointer to this type as its context parameter, like this:

    int

Then you can access the contents of this structure. For example, you can get the filename pointer as follows:

    bpf_probe_read_user_str

When you use a tracepoint program type, the structure passed to the eBPF program has already been mapped from a set of raw arguments. For better performance, you can directly access these raw arguments with a raw tracepoint eBPF program type. The section definition should start with `raw_tp` (or `raw_tracepoint`) instead of `tp`. You’ll need to convert the arguments from `__u64` to whatever type the tracepoint structure uses (when the tracepoint is the entry to a system call, these arguments are dependent on the chip architecture).