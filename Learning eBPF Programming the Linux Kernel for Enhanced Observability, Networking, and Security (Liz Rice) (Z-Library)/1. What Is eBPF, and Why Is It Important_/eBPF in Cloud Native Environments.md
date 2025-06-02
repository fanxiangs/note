# eBPF in Cloud Native Environments

[[null|]][[null|]]These days lots of organizations choose not to run applications by executing programs directly on servers. Instead, many use cloud native approaches: containers, orchestrators such as Kubernetes or ECS, or serverless approaches like Lambda, cloud functions, Fargate, and so on. These approaches all use automation to choose the server where each workload will run; in serverless, we’re not even aware what server is running each workload.

Nevertheless, there are servers involved, and each of those servers (whether it’s a virtual machine or bare-metal machine) runs a kernel. Where applications run in a container, if they’re running on the same (virtual) machine, they share the same kernel. In a Kubernetes environment, this means all the containers in all the pods on a given node are using the same kernel. When we instrument that kernel with eBPF programs, all the containerized workloads on that node are visible to those eBPF programs, as illustrated in [[#ebpf_programs_in_the_kernel_have_visibi|Figure 1-4]].

![eBPF programs in the kernel have visibility of all applications running on a Kubernetes node](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0104.png)

###### Figure 1-4. eBPF programs in the kernel have visibility of all applications running on a Kubernetes node

Visibility of all the processes on the node, combined with the ability to load eBPF programs dynamically, gives us the real superpowers of eBPF-based tooling in cloud native computing:

*   We don’t need to change our applications, or even the way they are configured, to instrument them with eBPF tooling.
    
*   As soon as it’s loaded into the kernel and attached to an event, an eBPF program can start observing preexisting application processes.

Contrast this with the _sidecar model_, which has been used to add functionality like logging, tracing, security, and service mesh functionality into Kubernetes apps. In the sidecar approach, the instrumentation runs as a container that is “injected” into each application pod. This process involves modifying the YAML that defines the application pods, adding in the definition of the sidecar container. This approach is certainly more convenient than adding the instrumentation into the source code of the application (which is what we had to do before the sidecar approach; for example, including a logging library in our application and making calls into that library at appropriate points in the code). Nevertheless, the sidecar approach has a few downsides:

*   The application pod has to be restarted for the sidecar to be added.
*   Something has to modify the application YAML. This is generally an automated process, but if something goes wrong, the sidecar won’t be added, which means the pod doesn’t get instrumented. For example, a deployment might be annotated to indicate that an admission controller should add the sidecar YAML to the pod spec for that deployment. But if the deployment isn’t labeled correctly, the sidecar won’t get added, and it’s therefore not visible to the instrumentation.
*   When there are multiple containers within a pod, they might reach readiness at different times, the ordering of which may not be predictable. Pod start-up time can be significantly slowed by the injection of sidecars, or worse, it can cause race conditions or other instabilities. For example, the [Open Service Mesh documentation](https://oreil.ly/z80Q5) describes how application containers have to be resilient to all traffic being dropped until the Envoy proxy container is ready.
*   Where networking functionality such as service mesh is implemented as a sidecar, it necessarily means that all traffic to and from the application container has to travel through the network stack in the kernel to reach a network proxy container, adding latency to that traffic; this is illustrated in [[#path_of_a_network_packet_using_a_networ|Figure 1-5]]. We’ll talk about improving network efficiency with eBPF in [[ch09.xhtml#ebpf_for_security|Chapter 9]].
    

![Path of a network packet using a service mesh proxy sidecar container](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0105.png)

###### Figure 1-5. Path of a network packet using a service mesh proxy sidecar container

All these issues are inherent problems with the sidecar model. Fortunately, now that eBPF is available as a platform, we have a new model that can avoid these issues. Additionally, because eBPF-based tools can see everything that’s happening on a (virtual) machine, they are harder for bad actors to sidestep. For example, if an attacker manages to deploy a cryptocurrency mining app on one of your hosts, they probably won’t do you the courtesy of instrumenting it with the sidecars you’re using on your application workloads. If you’re relying on a sidecar-based security tool to prevent apps from making unexpected network connections, that tool isn’t going to spot the mining app connecting to its mining pool if the sidecar isn’t injected. In contrast, network security implemented in eBPF can police all traffic on the host machine, so this cryptocurrency mining operation could easily be stopped. The ability to drop network packets for security reasons is something we’ll come back to in [[ch08.xhtml#ebpf_for_networking|Chapter 8]][[null|]].[[null|]][[null|]]