# BTF Information in the Object File

[[null|]][[null|]]The [kernel documentation for BTF](https://oreil.ly/5QrBy) describes how BTF data is encoded in an ELF object file in two sections: _.BTF_, which contains the data and string information, and _.BTF.ext_, which covers function and line information. You can use `readelf` to see that these sections have been added to the object file, like this:

$ readelf -S hello-buffer-config.bpf.o | grep BTF
  \[10\] .BTF              PROGBITS         0000000000000000  000002c0
  \[11\] .rel.BTF          REL              0000000000000000  00000e50
  \[12\] .BTF.ext          PROGBITS         0000000000000000  00000b18
  \[13\] .rel.BTF.ext      REL              0000000000000000  00000ea0

The `bpftool` utility lets us examine the BTF data from an object file, like this:

bpftool btf dump file hello-buffer-config.bpf.o

The output looks just like the output you get from dumping BTF info from loaded programs and maps, as you saw earlier in this chapter.

Let’s see how this BTF information can be used to allow the program to run on another machine with a different kernel version and different data structures.[[null|]][[null|]]