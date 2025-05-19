# Checking Pointers Before Dereferencing Them

[[null|]][[null|]][[null|]]One easy way to make a C program crash is to dereference a pointer when the pointer has a zero value (also known as _null_). Pointers indicate where in memory a value is being held, and zero is not a valid memory location. The eBPF verifier requires all pointers to be checked before they are dereferenced so that this type of crash can’t happen.

The example code in _hello-verifier.bpf.c_ looks for a custom message that might exist in the `my_config` hash table map for a user, with the following line:

    p

If there’s no entry in this map corresponding to `uid`, this will set `p` (which is a pointer to the message structure `msg_t`) to zero. Here’s a little bit of additional code that attempts to dereference this potentially null pointer:

    char

This compiles fine, but the verifier rejects it as follows:

; p = bpf\_map\_lookup\_elem(&my\_config, &uid); 
25: (18) r1 = 0xffff263ec2fe5000
27: (85) call bpf\_map\_lookup\_elem#1
28: (bf) r7 = r0                                [[#list_id_6_7|]]
; char a = p->message\[0\];
29: (71) r3 = \*(u8 \*)(r7 +0)                    [[#list_id_6_8|]]
R7 invalid mem access 'map\_value\_or\_null'

[[#code_id_6_7|]]

The return value from a helper function call gets stored in Register 0. Here, that value is being stored in Register 7. This means Register 7 now holds the value of the local variable `p`.

[[#code_id_6_8|]]

This instruction attempts to dereference the pointer value `p`. The verifier has been keeping track of the state of Register 7 and knows that it may hold a pointer to a map value, or it might be null.

The verifier rejects this attempt to dereference a null pointer, but the program will pass if there is an explicit check, like this:

    if

Some helper functions incorporate the pointer check for you. For example, if you look at the manpage for bpf-helpers, you’ll find the function signature for `bpf_probe_read_kernel()` is as follows:

    long

The third argument to this function is called `unsafe_ptr`. This is an example of a BPF helper function that helps programmers write safe code by handling checks for you. You’re allowed to pass a potentially null pointer—but only as the third argument called `unsafe_ptr`—and the helper function will check that it’s not null before attempting to deference it.