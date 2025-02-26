# Exercises

Here are a few things you can do to further explore BTF, CO-RE, and _libbpf_:

1.  Experiment with `bpftool btf dump map` and `bpftool btf dump prog` to see the BTF information associated with maps and programs, respectively. Remember that you can specify individual maps and programs in more than one way.
    
2.  Compare the output from `bpftool btf dump file` and `bpftool btf dump prog` for the same program in its ELF object file form and after it has been loaded into the kernel. They should be identical.
    
3.  Examine the debug output from _bpftool -d prog load hello-buffer-config.bpf.o /sys/fs/bpf/hello_. You’ll see each section being loaded, checks on the license, and relocations taking place, as well as output describing each BPF program instruction.
    
4.  Try building a BPF program against a different _vmlinux_ header file from BTFHub, and look in the debug output from `bpftool` for relocations that change offsets.
    
5.  Modify the _hello-buffer-config.c_ program so that you can configure different messages for different user IDs using the map (similar to the _hello-buffer-config.py_ example in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]]).
    
6.  Try changing the section name in the `SEC();`, perhaps to your own name. When you come to load the program into the kernel you should see an error because _libbpf_ doesn’t recognize the section name. This illustrates how _libbpf_ uses the section name to work out what kind of BPF program this is. You could try writing your own attachment code to explicitly attach to an event of your choice rather than relying on _libbpf_’s auto-attachment.
    

[^1] Strictly speaking, the data structure definitions come from kernel header files, and you could choose to compile based on a set of these header files that is different from what was used to build the kernel running on that machine. To work correctly (without the CO-RE mechanisms described in this chapter), the kernel headers have to be compatible with the kernel on the target machine where the eBPF program will run.

[^2] Part of this section is adapted from “What Is eBPF?” by Liz Rice. Copyright © 2022 O’Reilly Media. Used with permission.

[^3] A small and unscientific survey suggests that most people pronounce this the same as the word _core_ rather than in two syllables.

[^4] See the kernel documentation at [https://docs.kernel.org/bpf/btf.xhtml#type-encoding](https://docs.kernel.org/bpf/btf.xhtml#type-encoding).

[^5] The kernel needs to have been built with the `CONFIG_DEBUG_INFO_BTF` option enabled.

[^6] Which is the oldest Linux kernel version that can support BTF? See [https://oreil.ly/HML9m](https://oreil.ly/HML9m).

[^7] Well, normal C preprocessing applies so that you can do things like `#define`. But there’s no _special_ rewriting like there is when you use BCC.

[^8] eBPF programs handling network packets don’t get to use this helper function and can only access the network packet memory.

[^9] It is permitted in certain BTF-enabled program types such as `tp_btf`, `fentry`, and `fexit`.