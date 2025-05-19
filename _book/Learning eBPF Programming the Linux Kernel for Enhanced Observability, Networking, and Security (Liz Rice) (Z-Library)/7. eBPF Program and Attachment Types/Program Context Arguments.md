# Program Context Arguments

[[null|]][[null|]]All eBPF programs take a context argument that is a pointer, but the structure it points to depends on the type of event that triggered it. eBPF programmers need to write programs that accept the appropriate type of context; there is no point in pretending that the context argument points to a network packet if the event is, say, a tracepoint. Defining different types of programs allows the verifier to ensure that the contextual information is handled appropriately and to enforce rules about what helper functions are permissible.

###### Note

To dive into the details of the context data passed to different BPF program types, check out [this post by Alan Maguire on Oracle’s blog](https://oreil.ly/6dNIW).