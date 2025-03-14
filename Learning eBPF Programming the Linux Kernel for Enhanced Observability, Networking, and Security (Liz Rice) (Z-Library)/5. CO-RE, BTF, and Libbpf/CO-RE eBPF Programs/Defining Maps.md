# Defining Maps

[[null|]][[null|]]After including the header files, the next few lines of the source code in _hello-buffer-config.bpf.c_ define the structures used for maps, like this:

    struct

This requires more lines of code than I needed in the equivalent BCC example! With BCC, the map called `config` was created with the following macro:

    BPF_HASH

This macro isn’t available when you’re not using BCC, so in C you have to write it out longhand. You’ll see that I have used `__uint` and `__type`. These are defined in [bpf/bpf_helpers_def.h](https://oreil.ly/2FgjB) along with `__array`, like this:

    #

These macros generally seem to be used by convention in _libbpf_\-based programs, and I think they make the map definitions a little easier to read.

##### Note

The name “config” clashed with a definition in _vmlinux.h_, so I renamed the map “my\_config” for this example.