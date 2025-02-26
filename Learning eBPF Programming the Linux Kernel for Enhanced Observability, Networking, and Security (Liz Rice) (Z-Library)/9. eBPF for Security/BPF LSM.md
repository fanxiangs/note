# BPF LSM

[[null|]][[null|]]The LSM interface provides a set of hooks that each occur just before the kernel is about to act on a kernel data structure. The function called by a hook can make a decision about whether to allow the action to go ahead. This interface was originally provided to allow security tools to be implemented in the form of [kernel modules](https://oreil.ly/mF_OD); [BPF LSM](https://oreil.ly/KzaMT) extends this so that eBPF programs can be attached to the same hook points, as shown in [[#with_lsm_bpfcomma_ebpf_programs_can_be_|Figure 9-3]].

![With LSM BPF, eBPF programs can be triggered by LSM hook events](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0903.png)

###### Figure 9-3. With LSM BPF, eBPF programs can be triggered by LSM hook events

There are hundreds of LSM hooks, and they’re pretty nicely [documented in the kernel source code](https://oreil.ly/dO8jT). To be clear, there isn’t a one-to-one mapping between syscalls and LSM hooks, but if a syscall has the potential to do something interesting from a security perspective, processing that syscall will trigger one or more of the hooks.

Here’s a trivial example of an eBPF program attached to an LSM hook. This example is called during the processing of a `chmod` command (“chmod” stands for “change modes” and is mostly used to change the access permissions on a file):

    SEC

This example simply traces out the name of the file and always returns `0`, but you can imagine a real implementation that would make use of the arguments to decide whether to allow this change of mode. Returning a nonzero value would deny permission to make this change, so the kernel wouldn’t go ahead with it. It’s worth noting that making policy checks entirely within the kernel like this is highly performant.

The `path` argument to `BPF_PROG()` is the kernel data structure representing the file, and the `mode` argument is the desired new mode value. You can see the name of the file being accessed from the field `path->dentry->d_iname`.

LSM BPF was added in kernel version 5.7, which means that (at least at the time of this writing) it’s not yet available on many supported Linux distributions, but I expect that over the next couple of years many vendors will develop security tooling that makes use of this interface. Before LSM BPF is made widely available, there is another possible approach, as used by the developers of Cilium Tetragon.[[null|]][[null|]]