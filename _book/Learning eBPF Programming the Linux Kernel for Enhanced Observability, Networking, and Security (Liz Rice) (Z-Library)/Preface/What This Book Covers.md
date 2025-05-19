# What This Book Covers

eBPF continues to evolve at quite a rapid pace, which makes it rather difficult to write a comprehensive reference that doesn’t constantly need updating. However, there are some fundamentals and basic principles that are unlikely to change significantly, and that’s what this book discusses.

[[ch01.xhtml#what_is_ebpf_and_why_is_it_importantque|Chapter 1]] sets the scene by describing why eBPF is so powerful as a technology and explaining how the ability to run custom programs in the operating system kernel enables so many exciting capabilities.

Things become more concrete in [[ch02.xhtml#ebpfapostrophes_quotation_markhello_wor|Chapter 2]], where you’ll see some “Hello World” examples that introduce you to the concepts of eBPF programs and maps.

[[ch03.xhtml#anatomy_of_an_ebpf_program|Chapter 3]] dives into more detail about eBPF programs and how they run in the kernel, and [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]] explores the interface between user space applications and eBPF programs.

One of the big challenges of eBPF in recent years has been the question of compatibility across kernel versions. [[ch05.xhtml#co_recomma_btfcomma_and_libbpf|Chapter 5]] looks at the “compile once, run everywhere” (CO-RE) approach that solves this problem.

The verification process is perhaps the most important characteristic that distinguishes eBPF from kernel modules. I’ll introduce you to the eBPF verifier in [[ch06.xhtml#the_ebpf_verifier|Chapter 6]].

In [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]] you’ll get an introduction to the many different types of eBPF programs and their attachment points. Many of those attachment points are within the networking stack, and [[ch08.xhtml#ebpf_for_networking|Chapter 8]] explores the application of eBPF for networking features in more detail. [[ch09.xhtml#ebpf_for_security|Chapter 9]] looks at how eBPF is being used to build security tools.

If you want to write a user space application that interacts with eBPF programs, there are many libraries and frameworks available to help. [[ch10.xhtml#ebpf_programming|Chapter 10]] gives an overview of the options for various programming languages.

Finally, in [[ch11.xhtml#the_future_evolution_of_ebpf|Chapter 11]] I’ll gaze into my crystal ball and tell you about some future developments that are likely to unfold in the eBPF world.