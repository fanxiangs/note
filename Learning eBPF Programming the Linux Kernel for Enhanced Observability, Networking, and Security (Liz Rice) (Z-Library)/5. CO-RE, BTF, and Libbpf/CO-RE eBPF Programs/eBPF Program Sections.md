# eBPF Program Sections

[[null|]][[null|]][[null|]][[null|]]Use of _libbpf_ requires each eBPF program to be marked with a `SEC()` macro that defines the program type, like this:

    SEC

This results in a section called `kprobe` in the compiled ELF object, so _libbpf_ knows to load this as a `BPF_PROG_TYPE_KPROBE`. We’ll discuss different program types further in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].

Depending on the program type, you can also use the section name to specify what event the program will be attached to. The _libbpf_ library will use this information to set up the attachment automatically, rather than leaving you to do it explicitly in your user space code. So, for example, to auto-attach to the kprobe for the `execve` syscall on an ARM-based machine, you could specify the section like this:

    SEC

This requires you to know the function name for the syscall on that architecture (or figure it out, perhaps by looking at the _/proc/kallsyms_ file on your target machine, which lists all the kernel symbols, including its function names). But _libbpf_ can make life even easier for you with the `k(ret)syscall` section name, which tells the loader to attach to the kprobe in the architecture-specific function automatically:

    SEC

##### Note

The valid section names and formats are listed in the [libbpf documentation](https://oreil.ly/FhHrm). In the past, the requirements for section names were much looser, so you may come across eBPF programs written before _libbpf 1.0_ with section names that don’t match the valid set. Don’t let them confuse you!

The section definition declares where the eBPF program should be attached, and then the program itself follows. As before, the eBPF program itself is written as a C function. In the example code it’s called `hello()`, and it’s extremely similar to the `hello()` function you saw in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]]. Let’s consider the differences between that previous version and the version here:

    SEC

[[#code_id_5_1|]]

I’ve taken advantage of a [BPF_KPROBE_SYSCALL](https://oreil.ly/pgI1B) macro defined in _libbpf_ that makes it easy to access the arguments to a syscall by name. For `execve()`, the first argument is the pathname for the program that’s going to be executed. The eBPF program name is `hello`.

[[#code_id_5_2|]]

Since the macro has made it so easy to access that pathname argument to `execve()`, I’m including it in the data sent to the perf buffer output. Notice that copying memory requires the use of a BPF helper function.

[[#code_id_5_3|]]

Here, `bpf_map_lookup_elem()` is the BPF helper function for looking up values in a map, given a key. BCC’s equivalent of this would be `p = my_config.lookup(&data.uid)`. BCC rewrites this to use the underlying `bpf_map_lookup_elem()` function before it passes the C code to the compiler. When you’re using _libbpf_, there is no rewriting of the code before compilation,[^7] so you have to write directly to the helper functions.

[[#code_id_5_4|]]

Here’s another similar example where I have written directly to the helper function `bpf_perf_event_output()`, where BCC gave me the convenient equivalent `output.perf_submit(ctx, &data, sizeof(data))`.

The only other difference is that in the BCC version, I defined the message string as a local variable within the `hello()` function. BCC doesn’t (at least at the time of this writing) support global variables. In this version I have defined it as a global variable, like this:

    char

In _chapter4/hello-buffer-config.py_ the `hello` function was defined rather differently, like this:

    int

The `BPF_KPROBE_SYSCALL` macro is one of the convenient additions from _libbpf_ that I mentioned. You’re not required to use the macro, but it makes life easier. It does all the heavy lifting to provide named arguments for all the parameters passed to a syscall. In this case, it supplies a `pathname` argument that points to a string holding the path of the executable that is about to be run, which is the first argument to the `execve()` syscall.

If you’re paying very close attention you might notice that the `ctx` variable isn’t visibly defined in my source code for _hello-buffer-config.bpf.c_, but nevertheless, I’ve been able to use it when submitting data to the output perf buffer, like this:

    bpf_perf_event_output

The `ctx` variable does exist, hidden within the `BPF_KPROBE_SYSCALL` macro definition inside [bpf/bpf_tracing.h](https://oreil.ly/pgI1B), in _libbpf_, where you’ll also find some commentary about this. It can be slightly confusing to use a variable that’s not visibly defined, but it’s very helpful that it can be accessed.[[null|]][[null|]][[null|]]