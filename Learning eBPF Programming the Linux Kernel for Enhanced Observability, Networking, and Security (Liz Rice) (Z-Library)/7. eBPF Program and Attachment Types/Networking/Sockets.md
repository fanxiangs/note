# Sockets

[[null|]][[null|]]At the top of the stack, a subset of these network-related program types relates to sockets and socket operations:

*   `BPF_PROG_TYPE_SOCKET_FILTER` was the first program type to be added to the kernel. You probably guessed from the name that this is used for socket filtering, but what’s less obvious is that this doesn’t mean filtering data being sent to or from an application. It’s used to filter a _copy_ of socket data that can be sent to an observability tool such as tcpdump.
    
*   A socket is specific to a Layer 4 (TCP) connection. `BPF_PROG_TYPE_SOCK_OPS` allows eBPF programs to intercept various operations and actions that take place on a socket, and to set for that socket parameters such as TCP timeout values. Sockets only exist at the endpoints for a connection, and not on any middleboxes that they might pass through.
    
*   `BPF_PROG_TYPE_SK_SKB` programs are used in conjunction with a special map type that holds a set of references to sockets to provide what’s known as [sockmap operations](https://oreil.ly/0Enuo): redirecting traffic to different destinations at the socket layer.