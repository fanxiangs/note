# eBPF Is a Platform, Not a Feature

Coming up to a decade ago, the hot new technology was containers, and it seemed as though everyone was talking about what they were and what advantages they would bring. We’re at a similar stage with eBPF today, with lots of conference talks and blog posts—several of which I’ve referred to in this book—extolling the benefits of eBPF. Today, containers are part of daily life for many developers, whether they’re running code locally using Docker or other container runtimes, or deploying code to Kubernetes environments. Will eBPF become part of everyone’s regular toolkit too?

The answer, I believe, is no—or at least, not directly. Most users won’t write eBPF programs directly or manipulate them manually with utilities like `bpftool`. But they’ll interact regularly with tools built using eBPF, whether that’s for performance measurement, debugging, networking, security, tracing, or a whole host of other capabilities yet to be implemented using eBPF. Users might not be aware that they’re using eBPF, much as they might not know that when they use containers, they’re using kernel features like namespaces and cgroups.

Today, projects and vendors with knowledge of eBPF highlight their use of it because it’s so powerful and implies many advantages. As eBPF-based projects and products gain traction and market share, eBPF is becoming the de facto default technology platform for infrastructure tooling.

Knowledge of eBPF programming is—and will continue to be—a sought-after but relatively rare skill, just as kernel development today is much less common than developing, say, business applications or games. If you enjoy diving into the lower levels of systems and want to build essential infrastructure tooling, eBPF skills will serve you well. I hope this book has been of some assistance on your eBPF journey!

##### Further Reading

Throughout this book I have provided references to specific articles and documentation pages. Here is a list of some additional resources to help you on your eBPF journey:

*   The eBPF community site [ebpf.io](http://ebpf.io)
    
*   The BPF and XDP reference in the [Cilium documentation](https://docs.cilium.io)
    
*   [Linux kernel documentation on BPF](https://oreil.ly/q8xh3)
    
*   [Brendan Gregg’s website](https://www.brendangregg.com) on using eBPF for performance and observability
    
*   [Andrii Nakryiko’s website](https://nakryiko.com), particularly for more information on CO-RE and _libbpf_
    
*   [Lwn.net](https://lwn.net), a wonderful resource for updates to the Linux kernel, including the BPF subsystem
    
*   [Elixir.bootlin.com](http://elixir.bootlin.com), where you can browse the Linux source code
    
*   [eCHO](https://oreil.ly/2AATZ), a weekly livestream covering topics from across the eBPF and Cilium community (in which this author is a regular presenter)