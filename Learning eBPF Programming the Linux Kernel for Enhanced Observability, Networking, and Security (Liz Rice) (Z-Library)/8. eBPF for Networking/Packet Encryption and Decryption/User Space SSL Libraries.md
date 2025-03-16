# User Space SSL Libraries

[[null|]][[null|]]One common way to trace out the decrypted content of encrypted packets is to hook into calls made to user space libraries like OpenSSL or BoringSSL. An application using OpenSSL sends data to be encrypted by making a call to a function called `SSL_write()` and retrieves cleartext data that was received over the network in encrypted form using `SSL_read()`. Hooking eBPF programs into these functions with uprobes allows an application to observe the data _from any application that uses this shared library_ in the clear, before it is encrypted or after it has been decrypted. And there is no need for any keys, because those are already being provided by the application.

There is a fairly straightforward example called [openssl-tracer in the Pixie project](https://oreil.ly/puDp9),[^8] within which the eBPF programs are in a file called _openssl\_tracer\_bpf\_funcs.c_. Here’s the part of that code that sends data to user space, using a perf buffer (similar to examples you have seen earlier in this book):

    static

You can see that data from `buf` gets read into an `event` structure using the helper function `bpf_probe_read()`, and then that `event` structure is submitted to a perf buffer.

If this data is being sent to user space, it’s reasonable to assume this must be the data in unencrypted format. So where is this buffer of data obtained? You can work that out by seeing where the `process_SSL_data()` function is called. It’s called in two places: one for data being read and one for data being written. [[#ebpf_programs_are_hooked_to_uprobes_at_|Figure 8-4]] illustrates what is happening in the case of reading data that arrives on this machine in encrypted form.

When you’re reading data, you supply a pointer to a buffer to `SSL_read()`, and when the function returns, that buffer will contain the unencrypted data. Much like kprobes, the input parameters to a function—including that buffer pointer—are only available to a uprobe attached to the entry point, as the registers they’re held in might well get overwritten during the function’s execution. But the data won’t be available in the buffer until the function exits, when you can read it using a uretprobe.

![eBPF programs are hooked to uprobes at the entry to and exit from SSL_read() so that the unencrypted data can be read from the buffer pointer](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0804.png)

##### Figure 8-4. eBPF programs are hooked to uprobes at the entry to and exit from `SSL_read()` so that the unencrypted data can be read from the buffer pointer

So this example follows a common pattern for kprobes and uprobes, illustrated in [[#ebpf_programs_are_hooked_to_uprobes_at_|Figure 8-4]], where the entry probe temporarily stores input parameters using a map, from which the exit probe can retrieve them. Let’s look at the code that does this, starting with the eBPF program attached to the start of `SSL_read()`:

    // Function signature being probed:
    

[[#code_id_8_21|]]

As described in the comment for this function, the buffer pointer is the second parameter passed into the `SSL_read()` function to which this probe will be attached. The `PT_REGS_PARM2` macro gets this parameter from the context.

[[#code_id_8_22|]]

The buffer pointer is stored in a hash map, for which the key is the current process and thread ID, obtained at the start of the function using the helper `bpf_get_current_pid_tgif()`.

Here’s the corresponding program for the exit probe:

    int

[[#code_id_8_23|]]

Having looked up the current process and thread ID, use this as the key to retrieve the buffer pointer from the hash map.

[[#code_id_8_24|]]

If this isn’t a null pointer, call `process_SSL_data()`, which is the function you saw earlier that sends the data from that buffer to user space using the perf buffer.

[[#code_id_8_25|]]

Clean up the entry in the hash map, since every entry call should be paired with an exit.

This example shows how to trace out the cleartext version of encrypted data that gets sent and received by a user space application. The tracing itself is attached to a user space library, and there’s no guarantee that every application will use a given SSL library. The BCC project includes a utility called [sslsniff](https://oreil.ly/tFT9p) that also supports GnuTLS and NSS. But if someone’s application uses some other encryption library (or even, heaven forbid, they chose to “roll their own crypto”), the uprobes simply won’t have the right places to hook to and these tracing tools won’t work.

There are even more common reasons why this uprobe-based approach might not be successful. Unlike the kernel (of which there is only one per \[virtual\] machine), there can be multiple copies of user space library code. If you’re using containers, each one is likely to have its own set of all library dependencies. You can hook into uprobes in these libraries, but you’d have to identify the right copy for the particular container you want to trace. Another possibility is that rather than using a shared, dynamically linked library, an application might be statically linked so that it’s a single standalone executable[[null|]][[null|]].[[null|]][[null|]][[null|]][[null|]][[null|]][[null|]]