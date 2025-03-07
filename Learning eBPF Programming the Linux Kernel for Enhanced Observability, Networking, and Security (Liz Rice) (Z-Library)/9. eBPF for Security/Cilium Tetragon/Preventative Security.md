# Preventative Security

[[null|]][[null|]]Most eBPF-based security tools have used eBPF programs to detect malicious events, which notify a user space application that can then take action. As you can see in [[#an_asynchronous_notification_from_kerne|Figure 9-4]], any action the user space app takes happens asynchronously, by which time it might be too late—perhaps data could have been exfiltrated, or the attacker could have persisted malicious code onto disk.

![An asynchronous notification from kernel to user space allows some time for an attack to continue](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0904.png)

##### Figure 9-4. An asynchronous notification from kernel to user space allows some time for an attack to continue

In kernel versions 5.3 and up, there is a BPF helper function called `b⁠p⁠f⁠_⁠s⁠e⁠n⁠d⁠_​s⁠i⁠g⁠n⁠a⁠l⁠(⁠)`. Tetragon uses this function to implement preventative security. If a policy defines a Sigkill action, any matching events will cause Tetragon eBPF code to generate a SIGKILL signal that terminates the process that was attempting the out-of-policy action. As shown in [[#tetragon_kills_malicious_processes_sync|Figure 9-5]], this happens synchronously; that is, the activity the kernel was performing that the eBPF code determined to be out of policy is prevented from completing.

![Tetragon kills malicious processes synchronously by sending a SIGKILL signal from the kernel](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0905.png)

##### Figure 9-5. Tetragon kills malicious processes synchronously by sending a SIGKILL signal from the kernel

Sigkill policies need to be used with care, because an incorrectly configured policy could result in terminating applications unnecessarily, but it’s an incredibly powerful use of eBPF for security purposes. You can start by running in an “audit” mode that generates security events but doesn’t apply the SIGKILL enforcement, until you’re confident that the policy won’t break anything.

If you’re interested in learning more about using Cilium Tetragon for detecting security events, there is a report titled “[Security Observability with eBPF](https://www.oreilly.com/library/view/security-observability-with/9781492096719/)” by Natália Réka Ivánkó and Jed Salazar that digs into much more detail.[[null|]][[null|]]