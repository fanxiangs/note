# Initializing the Perf Buffer

[[null|]][[null|]][[null|]]You have seen the `bpf(BPF_MAP_UPDATE_ELEM)` calls that add entries into the `config` map. Next, the output shows some calls that look like this:

bpf(BPF\_MAP\_UPDATE\_ELEM, {map\_fd=4, key=0xffffa7842490, value=0xffffa7a2b410,
flags=BPF\_ANY}, 128) = 0

These look very similar to the calls that defined the `config` map entries, except in this case the map’s file descriptor is `4`, which represents the `output` perf buffer map.

As before, the key and the value are pointers, so you can’t tell the numeric value of either the key or the value from this `strace` output. I see this syscall repeated four times with identical values for all the parameters, though there’s no way of knowing whether the values the pointers hold have changed between each call. Looking at these `BPF_MAP_UPDATE_ELEM bpf()` calls leaves some unanswered questions about how the buffer is set up and used:

*   Why are there four calls to `BPF_MAP_UPDATE_ELEM`? Does this relate to the fact that the `output` map was created with a maximum of four entries?
    
*   After these four instances of `BPF_MAP_UPDATE_ELEM`, no more `bpf()` syscalls appear in the `strace` output. That might seem a little odd, because the map is there so that the eBPF program can write data every time it is triggered, and you’ve seen data being displayed by the user space code. That data is clearly not being retrieved from the map with `bpf()` syscalls, so how is it obtained?
    

You’ve also yet to see any evidence of how the eBPF program is getting attached to the kprobe event that triggers it. To get the explanation for all these concerns, I need `strace` to show a few more syscalls when running this example, like this:

$ strace -e bpf,perf\_event\_open,ioctl,ppoll ./hello-buffer-config.py

For brevity, I’m going to ignore calls to `ioctl()` that aren’t specifically related to the eBPF functionality of this example.