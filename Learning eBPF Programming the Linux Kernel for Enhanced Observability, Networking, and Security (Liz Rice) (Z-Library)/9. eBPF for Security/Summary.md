# Summary

In this chapter you saw how eBPF’s use in security has evolved from low-level checks on system calls to much more sophisticated use of eBPF programs for security policy checks, in-kernel event filtering, and runtime enforcement.

There’s still much active development in the area of using eBPF for security purposes. I believe we will see tools in this area evolving and becoming widely adopted over the coming years.[[null|]]

[^1] See, for example, this post from Jess Frazelle, who developed the default seccomp profile for Docker: [“How to Use the New Docker Seccomp Profiles”](https://oreil.ly/EcpnM).

[^2] The documentation for Inspektor Gadget’s seccomp profiler is quite dry, but [this video overview from Jose Blanquicet](https://oreil.ly/0bYaa) is more accessible.

[^3] Exploiting this window was discussed in a DEFCON 29 talk titled “[Phantom Attack: Evading System Call Monitoring](https://oreil.ly/WguKq)” by Rex Guo and Junyuan Zeng, and its impact on Falco was covered in more detail in the talk “[LSM BPF Change Everything](https://oreil.ly/17c-3)” by Leo Di Donato and KP Singh.