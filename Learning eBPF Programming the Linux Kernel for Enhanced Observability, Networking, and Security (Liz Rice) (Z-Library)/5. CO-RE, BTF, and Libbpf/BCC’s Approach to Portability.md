# BCC’s Approach to Portability

[[null|]]In [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]] I used [BCC](https://oreil.ly/ReUtn) to show a basic “Hello World” example of an eBPF program. The BCC project was the first popular project for implementing eBPF programs, providing a framework for both the user space and kernel aspects that’s relatively accessible to programmers without much kernel experience. To address portability across kernels, BCC took the approach of compiling eBPF code at runtime, in situ on the destination machine. There are a number of issues with this approach:

*   The compilation toolchain needs to be installed on every destination machine where you want the code to run, as well as the kernel header files (which aren’t always present by default).
    
*   You have to wait for the compilation to complete before the tool starts, which could mean a delay of several seconds, every time the tool is launched.
    
*   If you’re running the tool on a large fleet of identical machines, repeating the compilation on each machine is a waste of compute resources.
    
*   Some BCC-based projects package their eBPF source code and the toolchain into a container image, which makes distribution to each machine easier. But it doesn’t solve the problem of ensuring that the kernel headers are present, and it can even mean more duplication if several of these BCC containers are installed on each machine.
    
*   Embedded devices might not have sufficient memory resources to run the compilation step.
    

Because of these issues, if you’re planning to embark on developing a significant new eBPF project, I would not recommend using this legacy BCC approach for it, especially if you’re planning to distribute it for others to use. In this book I’ve given some examples based on BCC because it’s a good approach for learning about the basic concepts of eBPF, particularly because the Python user space code is so compact and easy to read. It’s also a perfectly good choice if you’re more comfortable with it and you want to put something together quickly. But it’s not the best approach for serious modern eBPF development.

The CO-RE approach offers a much better solution to the problem of cross-kernel portability for eBPF programs.

###### Note

The BCC project at [github.com/iovisor/bcc](https://oreil.ly/ReUtn) includes a wide range of command-line tools for observing all sorts of information about how a Linux machine is behaving. The original versions located in the [tools](https://oreil.ly/fI4w_) directory are mostly implemented in Python using this legacy approach to portability that I have described in this section.

In BCC’s [libbpf-tools](https://oreil.ly/ke7yq) directory, you’ll find updated versions of these tools written in C that take advantage of _libbpf_ and CO-RE and that don’t suffer from the problems I’ve just listed. They are an incredibly useful set of utilities!