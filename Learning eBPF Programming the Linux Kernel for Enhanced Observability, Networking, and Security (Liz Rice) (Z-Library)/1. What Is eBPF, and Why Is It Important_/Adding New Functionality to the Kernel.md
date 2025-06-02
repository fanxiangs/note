# Adding New Functionality to the Kernel

[[null|]][[null|]]The Linux kernel is complex, with around 30 million lines of code at the time of this writing.[^7] Making a change to any codebase requires some familiarity with the existing code, so unless you’re a kernel developer already, this is likely to present a challenge.

Additionally, if you want to contribute your change upstream, you’ll be facing a challenge that isn’t purely technical. Linux is a general-purpose operating system, used in all manner of environments and circumstances. This means that if you want your change to become part of an official Linux release, it’s not simply a matter of writing code that works. The code has to be accepted by the community (and more specifically by Linus Torvalds, creator and main developer of Linux) as a change that will be for the greater good of all. This isn’t a given—only one-third of submitted kernel patches are accepted.[^8]

Let’s suppose you’ve figured out a good technical approach for intercepting the system call for opening files. After some months of discussion and some hard development work on your part, let’s imagine the change is accepted into the kernel. Great! But how long will it be until it arrives on everyone’s machines?

There’s a new release of the Linux kernel every two or three months, but even when a change has made it into one of these releases, it’s still some time away from being available in most people’s production environments. This is because most of us don’t just use the Linux kernel directly—we use Linux distributions like Debian, Red Hat, Alpine, and Ubuntu that package up a version of the Linux kernel with various other components. You may well find that your favorite distribution is using a kernel release that’s several years old.

[[null|]][[null|]]For example, a lot of enterprise users employ Red Hat Enterprise Linux (RHEL). At the time of this writing, the current release is RHEL 8.5, dated November 2021, and it uses version 4.18 of the Linux kernel. This kernel was released in August 2018.

As illustrated in the cartoon in [[#adding_features_to_the_kernel_left_pare|Figure 1-2]], it takes literally years to get new functionality from the idea stage into a production environment Linux kernel.[^9]

![Adding features to the kernel (cartoon by Vadim Shchekoldin, Isovalent)](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0102.png)

###### Figure 1-2. Adding features to the kernel (cartoon by Vadim Shchekoldin, Isovalent)