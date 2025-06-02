# Visualizing Control Flow

[[null|]]The verifier explores all the possible paths through the eBPF program, and if you’re trying to debug an issue, it can be helpful to see those paths for yourself. [[null|]]The `bpftool` utility can help with this by producing a control flow graph of the program in [DOT format](https://oreil.ly/V-1WN), which you can then convert into an image format, like this:

$ bpftool prog dump xlated name kprobe\_exec visual > out.dot
$ dot -Tpng out.dot > out.png

This produces a visual representation of the control flow like that shown in [[#extract_from_control_flow_graph_left_pa|Figure 6-1]].

![Extract from control flow graph (the full image can be found as chapter6/kprobe_exec.png in the GitHub repo for this book)](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0601.png)

###### Figure 6-1. Extract from control flow graph (the full image can be found as chapter6/kprobe\_exec.png in the [GitHub repo](http://github.com/lizrice/learning-ebpf) for this book)