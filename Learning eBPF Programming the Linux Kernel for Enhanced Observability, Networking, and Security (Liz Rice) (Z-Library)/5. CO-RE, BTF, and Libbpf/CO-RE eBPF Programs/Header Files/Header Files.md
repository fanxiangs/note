# Header Files

[[null|]][[null|]]The first few lines of _hello-buffer-config.bpf.c_ specify the header files that it needs:

#include "vmlinux.h"
#include 
#include 
#include 
#include "hello-buffer-config.h"

These five files are the _vmlinux.h_ file, a few headers from _libbpf_, and an application-specific header file that I wrote myself. Let’s see why this is a typical pattern for the header files needed for a _libbpf_ program.