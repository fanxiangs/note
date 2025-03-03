# Bpftrace

[[null|]][[null|]]As described on the project’s _README_ page, “`bpftrace` is a high-level tracing language for Linux eBPF … inspired by awk and C, and predecessor tracers such as DTrace and SystemTap.”

The [bpftrace](https://oreil.ly/BZNZO) command-line tool converts programs written in this high-level language into eBPF kernel code and provides some output formatting for the results within the terminal. As a user, you don’t really need to think about the kernel–user space split.

You’ll find several examples of useful one-liners in the project documentation, including a nice [tutorial](https://oreil.ly/Ah2QB) that takes you from writing a simple “Hello World” script up to writing more sophisticated scripts that can trace out data read from within kernel data structures.

###### Note

Get a feel for the range of capabilities that `bpftrace` provides from Brendan Gregg’s [bpftrace cheat sheet](https://oreil.ly/VBwLm). Or, for in-depth coverage of both `bpftrace` and BCC, see his book [BPF Performance Tools](https://oreil.ly/kjc95).

As its name suggests, `bpftrace` can attach to tracing (also known as perf-related) events, including kprobes, uprobes, and tracepoints. For example, you can list the available tracepoints and kprobes on a machine with the `-l` option, like this:

$ bpftrace -l "\*execve\*"
tracepoint:syscalls:sys\_enter\_execve
tracepoint:syscalls:sys\_exit\_execve
...
kprobe:do\_execve\_file
kprobe:do\_execve
kprobe:\_\_ia32\_sys\_execve
kprobe:\_\_x64\_sys\_execve
...

This example finds all the available attachment points that contain “execve.” From this output you can see that it’s possible to attach to a kprobe called `do_execve`. Here’s a `bpftrace` one-line script that attaches to that event:

bpftrace -e 'kprobe:do\_execve { @\[comm\] = count(); }'
Attaching 1 probe...
^C

@\[node\]: 6
@\[sh\]: 6
@\[cpuUsage.sh\]: 18

The `{ @[comm] = count(); }` part is the script attached to that event. This example keeps track of the number of times the event was triggered by different executables.

Scripts for `bpftrace` can coordinate multiple eBPF programs attached to different events. For example, consider the [opensnoop.bt script](https://oreil.ly/3HWZ2) that reports on files being opened. Here is an extract:

tracepoint:syscalls:sys\_enter\_open,
tracepoint:syscalls:sys\_enter\_openat
{
    @filename\[tid\] = args->filename;
}

tracepoint:syscalls:sys\_exit\_open,
tracepoint:syscalls:sys\_exit\_openat
/@filename\[tid\]/
{
    $ret = args->ret;
    $fd = $ret > 0 ? $ret : -1;
    $errno = $ret > 0 ? 0 : - $ret;

    printf("%-6d %-16s %4d %3d %s\\n", pid, comm, $fd, $errno,
        str(@filename\[tid\]));
    delete(@filename\[tid\]);
}

This script defines two different eBPF programs, each attached to two different kernel tracepoints, at the entry to and exit from the `open()` and `openat()` syscalls.[^1] Both of these syscalls are used to open files and take a filename as an input argument. The program triggered by either flavor of syscall entry caches that filename, storing it in a map where the key is the current thread ID. When the exit tracepoint is hit, the cached filename is retrieved from this map by the `/@filename[tid]/` line in the script.

Running this script generates output like this:

./opensnoop.bt 
Attaching 6 probes...
Tracing open syscalls... Hit Ctrl-C to end.
PID    COMM               FD ERR PATH
297388 node               30   0 /home/liz/.vscode-server/data/User/
                                 workspaceStorage/73ace3ed015
297360 node               23   0 /proc/307224/cmdline
297360 node               23   0 /proc/305897/cmdline
297360 node               23   0 /proc/307224/cmdline

I’ve just told you there are four eBPF programs attached to tracepoints, so why does this output say there are six probes? The answer is that there are two “special probes” for the `BEGIN` and `END` clauses that the full version of this program includes to initialize and clean up the script (very similar to the awk language). I’ve omitted those clauses here for brevity, but you can find them in the [source code in GitHub](https://oreil.ly/X8wgW).

If you’re using `bpftrace`, you don’t need to know about the underlying programs and maps, but for those of you who have read earlier chapters of this book, those concepts should be familiar to you by now. If you’re interested to see the programs and maps that are loaded in the kernel while a `bpftrace` program is running, you can easily do this with `bpftool` (just as you saw in [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]). Here’s the output I got while running _opensnoop.bt_:

$ bpftool prog list 
...
494: tracepoint  name sys\_enter\_open  tag 6f08c3c150c4ce6e  gpl
        loaded\_at 2022-11-18T12:44:05+0000  uid 0
        xlated 128B  jited 93B  memlock 4096B  map\_ids 254
495: tracepoint  name sys\_enter\_opena  tag 26c093d1d907ce74  gpl
        loaded\_at 2022-11-18T12:44:05+0000  uid 0
        xlated 128B  jited 93B  memlock 4096B  map\_ids 254
496: tracepoint  name sys\_exit\_open  tag 0484b911472301f7  gpl
        loaded\_at 2022-11-18T12:44:05+0000  uid 0
        xlated 936B  jited 565B  memlock 4096B  map\_ids 254,255
497: tracepoint  name sys\_exit\_openat  tag 0484b911472301f7  gpl
        loaded\_at 2022-11-18T12:44:05+0000  uid 0
        xlated 936B  jited 565B  memlock 4096B  map\_ids 254,255

$ bpftool map list 
254: hash  flags 0x0
        key 8B  value 8B  max\_entries 4096  memlock 331776B
255: perf\_event\_array  name printf  flags 0x0
        key 4B  value 4B  max\_entries 2  memlock 4096B

You can clearly see the four tracepoint programs, plus the hash map that’s used for caching the filenames and the `perf_event_array` that is being used to pass output data from kernel to user space.

###### Note

The `bpftrace` utility is built on top of BCC, which you met elsewhere in this book and which I’ll cover later in this chapter. `bpftrace` scripts get converted into BCC programs, which are then compiled at runtime using the LLVM/Clang toolchain.

If you want command-line tools for eBPF-based performance measurement, you may well find that your needs are met using [bpftrace](https://oreil.ly/u5FrJ). But although `bpftrace` can be a powerful tool for using eBPF for tracing, it doesn’t open up the full range of possibilities that eBPF enables.

To unlock the full potential of eBPF, you’ll need to directly write eBPF programs yourself for the kernel and also handle the user space part. These two aspects can, and often are, written in entirely different languages. Let’s start with the choices for eBPF code that runs in the kernel.[[null|]][[null|]]