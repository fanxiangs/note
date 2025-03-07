# Cilium Tetragon

[[null|]][[null|]][Tetragon](https://oreil.ly/p-bdc) is part of the Cilium project (also part of the CNCF). Rather than attaching to LSM API hooks, Tetragon’s approach is to build a framework for attaching eBPF programs to arbitrary functions in the Linux kernel.

Tetragon is designed for use in a Kubernetes environment, and the project defines a custom Kubernetes resource type called a _TracingPolicy_. This is used to define a set of events to which eBPF programs should be attached, conditions that need to be checked by eBPF code, and actions to take if the conditions are met. The following is an extract from a sample TracingPolicy:

spec:
 kprobes:
 - call: "fd\_install"
...
     matchArgs:
     - index: 1
       operator: "Prefix"
       values:
       - "/etc/"
...

This policy defines a set of kprobes to attach programs to, the first of which is the kernel function `fd_install`. This is an internal function in the kernel. Let’s explore why you might choose to attach to a function like that.