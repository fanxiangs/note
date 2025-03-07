# Kfuncs

[[null|]][[null|]]Kfuncs allow internal kernel functions to be registered with the BPF subsystem so that the verifier will allow them to be called from eBPF programs. There is a registration for each eBPF program type that is permitted to call a given kfunc.

Unlike helper functions, kfuncs don’t provide compatibility guarantees, so an eBPF programmer has to consider the possibility of changes between kernel versions.

There is a set of [“core” BPF kfuncs](https://oreil.ly/06qoi), which at the time of this writing consists of functions that allow eBPF programs to obtain and release kernel references to tasks and cgroups.

To recap, the type of an eBPF program determines what events it can be attached to, which in turn defines the type of context information it receives. The program type also defines the set of helper functions and kfuncs it can call.

Program types are broadly considered to fall into two categories: tracing (or perf) program types and networking-related program types. Let’s look at some examples.