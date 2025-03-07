# Managing an event buffer

[[null|]][[null|]][[null|]]Setting up the perf buffer uses a function defined in _libbpf_ itself, rather than in the skeleton:

    pb

You can see the `perf_buffer__new()` function takes the file descriptor for the “output” map as the first argument. The `handle_event` argument is a callback function that gets called when new data arrives in the perf buffer, and `lost_event` gets called if there isn’t enough room in the perf buffer for the kernel to write a data entry. In my example these functions just write messages to the screen.

Finally, the program has to poll the perf buffer repeatedly:

    while

The 100 is a timeout in milliseconds. The callback functions previously set up will get called as appropriate when data arrives or when the buffer is full.

Finally, to clean up I free the perf buffer and destroy the eBPF programs and maps in the kernel, like this:

    perf_buffer__free

There are a whole set of `perf_buffer_*`\- and `ring_buffer_*`\-related functions in _libbpf_ to help you manage event buffers.

If you make and run this example `hello-buffer-config` program, you’ll see the following output (that’s very similar to what you saw in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]]):[[null|]][[null|]][[null|]]

23664  501    bash             Hello World
23665  501    bash             Hello World
23667  0      cron             Hello World
23668  0      sh               Hello World