# Listing BTF Information with bpftool

[[null|]][[null|]]As with programs and maps, you can use the `bpftool` utility to show BTF information. The following command lists all the BTF data loaded into the kernel:

bpftool btf list
1: name \[vmlinux\]  size 5843164B
2: name \[aes\_ce\_cipher\]  size 407B
3: name \[cryptd\]  size 3372B
...
149: name   size 4372B  prog\_ids 319  map\_ids 103
        pids hello-buffer-co(7660)
155: name   size 37100B
        pids bpftool(7784)

(I’ve omitted many entries from the results for brevity.)

The first entry in the list is `vmlinux`, and it corresponds to the _vmlinux_ file I mentioned earlier that holds the BTF information about the currently running kernel.

##### Note

Some of the examples early in this chapter reuse the programs from [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]], and then later in this chapter you’ll find new examples for which the source is in the _chapter5_ directory at [github.com/lizrice/learning-ebpf](https://github.com/lizrice/learning-ebpf).

To obtain this example output I ran this command while the `hello-buffer-config` example from [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]] was running. You can see the entry describing the BTF information that this process is using, on the line that starts with `149:`:

149: name   size 4372B  prog\_ids 319  map\_ids 103
        pids hello-buffer-co(7660)

Here’s what that line is telling us:

*   This chunk of BTF information has ID 149.
    
*   It’s an anonymous blob of around 4 KB of BTF information.
    
*   It’s used by the BPF program with `prog_id 319` and the BPF map with `map_id 103`.
    
*   It’s also used by the process with ID 7660 (shown within parentheses) running the `hello-buffer-config` executable (whose name has been truncated to 15 characters).
    

These program, map, and BTF identifiers match with the following output that `bpftool` shows about `hello-buffer-config`’s program called `hello`:

bpftool prog show name hello
319: kprobe  name hello  tag a94092da317ac9ba  gpl
        loaded\_at 2022-08-28T14:13:35+0000  uid 0
        xlated 400B  jited 428B  memlock 4096B  map\_ids 103,104
        btf\_id 149
        pids hello-buffer-co(7660)

The only thing that doesn’t appear to match completely between these two sets of information is that the program refers to an extra `map_id`, `104`. That’s the perf event buffer map, and it doesn’t use BTF information; hence, it doesn’t appear in the BTF-related output.

Much like `bpftool` can dump the contents of programs and maps, it can also be used to view the BTF type information contained in a blob of data.[[null|]][[null|]]