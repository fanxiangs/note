# Detaching the Program

[[null|]][[null|]][[null|]][[null|]]You can detach the program from the network interface like this:

$ bpftool net detach xdp dev eth0

There is no output if this command runs successfully, but you can confirm that the program is no longer attached by the lack of XDP entries in the output from `bpftool net list`:

$ bpftool net list 
xdp:

tc:

flow\_dissector:

However, the program is still loaded into the kernel:

$ bpftool prog show name hello 
395: xdp  name hello  tag 9d0e949f89f1a82c  gpl
        loaded\_at 2022-12-19T18:20:32+0000  uid 0
        xlated 48B  jited 108B  memlock 4096B  map\_ids 4