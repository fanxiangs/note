# Application-specific headers

[[null|]]It’s very common to have an application-specific header file that defines any structures that are used by both the user space and eBPF parts of your app. In my example, the _hello-buffer-config.h_ header file defines the `data_t` structure that I’m using to pass event data from the eBPF program to user space. It’s almost the same structure you saw in the BCC version of this code, and it looks like this:

    struct

The only difference from the version you saw before is that I have added a field called `path`.

The reason to pull this structure definition into a separate header file is that I will also refer to it from the user space code in _hello-buffer-config.c_. In the BCC version, the kernel and user space code were both defined in a single file, and BCC did some work behind the scenes to make the structure available to the Python user space code.[[null|]][[null|]]