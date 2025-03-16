# Summary

When I first got interested in eBPF, getting code through the verifier seemed like a dark art, where seemingly valid code would get rejected, throwing up what seemed to be arbitrary errors. Over time there have been _lots_ of improvements to the verifier, and in this chapter you’ve seen several examples where the verifier log gives hints to help you figure out what the problem is.

These hints are more helpful when you have a mental model of how the eBPF (virtual) machine works, using a set of registers for temporary value storage as it steps through an eBPF program. The verifier keeps track of the types and possible range of values for each register to ensure that eBPF programs are safe to run.

If you try writing some eBPF code of your own, you might find yourself needing assistance to resolve verifier errors. The [eBPF community Slack channel](http://ebpf.io/slack) is a good place to ask for help, and lots of people have also found advice on [StackOverflow](https://oreil.ly/nu_0v).[[null|]]