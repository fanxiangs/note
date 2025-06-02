# Global Variables

[[null|]][[null|]][[null|]]As you learned in the previous chapter, an eBPF map is a data structure that can be accessed from an eBPF program or from user space. Since the same map can be accessed repeatedly by different runs of the same program, it can be used to hold state from one execution to the next. Multiple programs can also access the same map. Because of these characteristics, map semantics can be repurposed for use as global variables.

###### Note

Before support for [global variables was added in 2019](https://oreil.ly/IDftt), eBPF programmers had to write maps explicitly to perform the same task.

You saw earlier that `bpftool` shows this example program using two maps with the identities 165 and 166. (You will probably see different identities if you try this for yourself, as the identities are assigned when the maps are created in the kernel.) Let’s explore what is in those maps.

[[null|]]The `bpftool` utility can show the maps loaded into the kernel. For clarity I will only show the entries 165 and 166 that relate to the example “Hello World” program:

$ bpftool map list
165: array  name hello.bss  flags 0x400
        key 4B  value 4B  max\_entries 1  memlock 4096B
        btf\_id 254
166: array  name hello.rodata  flags 0x80
        key 4B  value 15B  max\_entries 1  memlock 4096B
        btf\_id 254  frozen

A bss[^6] section in an object file compiled from a C program typically holds global variables, and you can use `bpftool` to inspect its contents, like this:

$ bpftool map dump name hello.bss
\[{
        "value": {
            ".bss": \[{
                    "counter": 11127
                }
            \]
        }
    }
\]

I could also have used `bpftool map dump id 165` to retrieve the same information. If I run either of these commands again, I’ll see that the counter has increased, as the program has been run every time a network packet is received.

As you’ll learn in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], `bpftool` is able to pretty-print the field names from a map (here, the variable name `counter`) only if BTF information is available, and that information is included only if you compile with the `-g` flag. If you omitted that flag during the compilation step, you’d see something that looks more like this:

$ bpftool map dump name hello.bss
key: 00 00 00 00  value: 19 01 00 00
Found 1 element

Without BTF information, `bpftool` has no way of knowing what variable name was used in the source code. You can infer that since there is only one item in this map, the hex value `19 01 00 00` must be the current value of `counter` (281 in decimal, since the bytes are ordered starting with the least significant byte).

You’ve seen here that the eBPF program uses the semantics of a map to read and write to a global variable. Maps are also used to hold static data, as you can see by inspecting the other map.

The fact that the other map is named `hello.rodata` gives a hint that this could be read-only data related to our _hello_ program. You can dump the contents of this map to see that it holds the string used by the eBPF program for tracing:

$ bpftool map dump name hello.rodata
\[{
        "value": {
            ".rodata": \[{
                    "hello.\_\_\_\_fmt": "Hello World %d"
                }
            \]
        }
    }
\]

If you didn’t compile the object with the `-g` flag, you’ll see output that looks like this:

$ bpftool map dump id 166
key: 00 00 00 00  value: 48 65 6c 6c 6f 20 57 6f  72 6c 64 20 25 64 00
Found 1 element

There is one key–value pair in this map, and the value contains 12 bytes of data ending with a 0. It probably won’t surprise you that those bytes are the ASCII representation of the string `"Hello World %d"`.

Now that we’ve finished inspecting this program and its maps, it’s time to clean it up. We’ll start by detaching it from the event that triggers it.[[null|]][[null|]][[null|]]