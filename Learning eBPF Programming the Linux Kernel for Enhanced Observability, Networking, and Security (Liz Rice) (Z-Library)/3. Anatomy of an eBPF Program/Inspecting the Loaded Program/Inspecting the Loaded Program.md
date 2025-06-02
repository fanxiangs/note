# Inspecting the Loaded Program

[[null|]][[null|]][[null|]]The [[null|]]`bpftool` utility can list all the programs that are loaded into the kernel. If you try this yourself you’ll probably see several preexisting eBPF programs in this output, but for clarity I will just show the lines that relate to our “Hello World” example:

$ bpftool prog list 
...
540: xdp  name hello  tag d35b94b4c0c10efb  gpl
        loaded\_at 2022-08-02T17:39:47+0000  uid 0
        xlated 96B  jited 148B  memlock 4096B  map\_ids 165,166
        btf\_id 254

The program has been assigned the ID 540. This identity is a number assigned to each program as it’s loaded. Knowing the ID, you can ask `bpftool` to show more information about this program. This time, let’s get the output in prettified JSON format so that the field names are visible, as well as the values:

$ bpftool prog show id 540 --pretty
{
    "id": 540,
    "type": "xdp",
    "name": "hello",
    "tag": "d35b94b4c0c10efb",
    "gpl\_compatible": true,
    "loaded\_at": 1659461987,
    "uid": 0,
    "bytes\_xlated": 96,
    "jited": true,
    "bytes\_jited": 148,
    "bytes\_memlock": 4096,
    "map\_ids": \[165,166
    \],
    "btf\_id": 254
}

Given the field names, a lot of this is straightforward to understand:

*   The program’s ID is 540.
    
*   The `type` field tells us this program can be attached to a network interface using the XDP event. Several other types of BPF programs can be attached to different sorts of events, and we’ll discuss this more in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]].
    
*   The name of the program is `hello`, which is the function name from the source code.
    
*   The `tag` is another identifier for this program, which I’ll describe in more detail shortly.
    
*   The program is defined with a GPL-compatible license.
    
*   There’s a timestamp showing when the program was loaded.
    
*   User ID 0 (which is root) loaded the program.
    
*   There are 96 bytes of translated eBPF bytecode in this program, which I’ll show you shortly.
    
*   This program has been JIT-compiled, and the compilation resulted in 148 bytes of machine code. I’ll cover this shortly too.
    
*   The `bytes _memlock` field tells us this program reserves 4,096 bytes of memory that won’t be paged out.
    
*   This program refers to BPF maps with IDs 165 and 166. This might seem surprising, since there is no obvious reference to maps in the source code. You’ll see later in this chapter how map semantics are used to handle global data in eBPF programs.
    
*   You’ll learn about BTF in [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]], but for now just know that `btf_id` indicates there is a block of BTF information for this program. This information is included in the object file only if you compile with the `-g` flag.