# Flow Dissector

[[null|]][[null|]]A flow dissector is used at various points in the network stack to extract details from a packet’s headers. eBPF programs of type `BPF_PROG_TYPE_FLOW_DISSECTOR` can implement custom packet dissection. There’s a nice write-up in this LWN article on [writing network flow dissectors in BPF](https://oreil.ly/nFKLV).