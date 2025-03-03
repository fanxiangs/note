# Exercises

If you’d like to try one or more of the libraries discussed in this chapter, “Hello World” is always a good place to start:

1.  Using one or more libraries of your choosing, write an example “Hello World” program that outputs a simple trace message.
    
2.  Use `llvm-objdump` to compare the bytecode produced with the “Hello World” example from [[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]]. You’ll find lots of similarities!
    
3.  As you saw in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]], you can use `strace -e bpf` to see when `bpf()` system calls are made. Try that on your “Hello World” program to see if it’s behaving as you expect.
    

[^1] Being attached to syscall entry points means this script has the same Time Of Check To Time Of Use (TOCTOU) vulnerability discussed in the previous chapter. That doesn’t stop it from being a useful tool; it’s just that you shouldn’t rely on it as your only line of defense for security purposes.

[^2] For an example of this, check out Cloudflare’s blog post [“eBPF, Sockets, Hop Distance and manually writing eBPF assembly”](https://oreil.ly/2GjuK).

[^3] See, for example, Brendan Gregg’s [observation](https://oreil.ly/fz_dQ) that the _libbpf_\-based version of opensnoop required around 9 MB compared with 80 MB for the Python-based version.

[^4] Watch me working through some of the XDP Tutorial examples in [episode 13 of the eBPF and Cilium Office Hours” livestream](https://oreil.ly/9SaKn).

[^5] Dave Cheney’s 2016 post “[cgo is not Go](https://oreil.ly/mxThs)” remains a good overview of concerns related to the CGo boundary.

[^6] As well as the `bpftrace` version of this tool, there are equivalents in BCC and in _libbpf-tools_. They all do very much the same thing, generating a line of trace whenever a process opens a file. There’s a walkthrough of the eBPF code for BCC’s version of opensnoop in my report [“What Is eBPF?”](https://www.oreilly.com/library/view/what-is-ebpf/9781492097266).