# Loading programs and maps into the kernel

[[null|]][[null|]]The first call to an auto-generated function is this one:

    skel

As its name suggests, this function covers two phases: opening and loading. The “open” phase involves reading the ELF data and converting its sections into structures that represent eBPF programs and maps. The “load” phase loads those maps and programs into the kernel, performing any CO-RE fixups as necessary.

These two phases can easily be handled separately, as the skeleton code provides separate `name__open()` and `name__load()` functions. This gives you the option to manipulate the eBPF information before loading it. This is commonly done to configure a program before loading it. For example, I could initialize a counter global variable `c` to some value, like this:

    skel

The data type returned by `hello_buffer_config_bpf__open()`, and also by `hello_buffer_config_bpf__load()`, is a structure called `hello_buffer_config_bpf` defined in the skeleton header to include information about all the maps, programs, and data defined in the object file.

#### Note

The skeleton object (`hello_buffer_config_bpf` in this example) is just a user space representation of information from the ELF bytes. Once it has been loaded into the kernel, if you change a value in the object, it won’t have any effect on the kernel-side data. So, for example, changing `skel->data->c` after loading will not have any effect.