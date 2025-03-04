# Encrypted Connections

[[null|]][[null|]][[null|]]Many organizations have requirements to protect their deployments and their users’ data by encrypting traffic between applications. This can be achieved by writing code in each application to ensure that it sets up secure connections, typically using mutual Traffic Layer Security (mTLS) underpinning an HTTP or gRPC connection. Setting up these connections requires first establishing the identities of the apps at either end of the connection (which is usually achieved by exchanging certificates) and then encrypting the data that flows between them.

In Kubernetes, it’s possible to offload the requirement from the application, either to a service mesh layer or to the underlying network itself. A full discussion of service mesh is beyond the scope of this book, but you might be interested in a piece I wrote on the new stack: [“How eBPF Streamlines the Service Mesh”](https://oreil.ly/5ayvF). Let’s concentrate here on the network layer and how eBPF makes it possible to push the encryption requirement into the kernel.

[[null|]][[null|]]The simplest option to ensure that traffic is encrypted within a Kubernetes cluster is to use _transparent encryption_. It’s called “transparent” because it takes place entirely at the network layer and it’s extremely lightweight from an operational point of view. The applications themselves don’t need to be aware of the encryption at all, and they don’t need to set up HTTPS connections; nor does this approach require any additional infrastructure components running under Kubernetes.

[[null|]][[null|]]There are two in-kernel encryption protocols in common usage, IPsec and WireGuard(R), and they’re both supported in Kubernetes networking by Cilium and Calico CNIs. It’s beyond the scope of this book to discuss the differences between these two protocols, but the key point is that they set up a secure tunnel between two machines. The CNI can choose to connect the eBPF endpoint for a pod via this secure tunnel.

##### Note

There is a nice write-up on the [Cilium blog](https://oreil.ly/xjpGP) of how Cilium uses WireGuard(R) as well as IPsec to provide encrypted traffic between nodes. The post also gives a brief overview of the performance characteristics of both.

The secure tunnel is set up using the identities of the nodes at either end. These identities are managed by Kubernetes anyway, so the administrative burden for an operator is minimal. For many purposes this is sufficient as it ensures that all network traffic in a cluster is encrypted. Transparent encryption can also be used unmodified with NetworkPolicy that uses Kubernetes identities to manage whether traffic can flow between different endpoints in the cluster.

Some organizations operate a multitenant environment where there’s a need for strong multitenant boundaries and where it’s essential to use certificates to identify every application endpoint. Handling this within every application is a significant burden, so it’s something that more recently has been offloaded to a service mesh layer, but this requires a whole extra set of components to be deployed, causing additional resource consumption, latency, and operational complexity.

eBPF is now enabling a [new approach](https://oreil.ly/DSnLZ) that builds on transparent encryption but uses TLS for the initial certificate exchange and endpoint authentication so that the identities can represent individual applications rather than the nodes they are running on, as depicted in [[#transparent_encryption_between_authenti|Figure 8-8]].

![Transparent encryption between authenticated application identities](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0808.png)

##### Figure 8-8. Transparent encryption between authenticated application identities

Once the authentication step has taken place, IPsec or WireGuard(R) within the kernel is used to encrypt the traffic that flows between those applications. This has a number of advantages. It allows third-party certificate and identity management tools like cert-manager or SPIFFE/SPIRE to handle the identity part, and the network takes care of encryption so that it’s all entirely transparent to the application. Cilium supports NetworkPolicy definitions that specify endpoints by their SPIFFE ID rather than just by their Kubernetes labels. And perhaps most importantly, this approach can be used with any protocol that travels in IP packets. That’s a big step up from mTLS, which works only for TCP-based connections.

There’s not enough room in this book to dive deep into all the internals of Cilium, but I hope this section helped you understand how eBPF is a powerful platform for building complex networking functionality like a fully featured Kubernetes CNI[[null|]][[null|]][[null|]].[[null|]][[null|]]