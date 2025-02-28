# Hash Table Map

[[null|]][[null|]]Like the previous example in this chapter, this eBPF program will be attached to a kprobe at the entry to the `execve` system call. It’s going to populate a hash table with key–value pairs, where the key is a user ID and the value is a counter for the number of times `execve` is called by a process running under that user ID. In practice, this example will show how many times each different user has run programs.

First, let’s look at the C code for the eBPF program itself:

```c
BPF_HASH(counter_table);                                     

int hello(void *ctx) {
  u64 uid;                                                  
  u64 counter = 0;
  u64 *p;

  uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;              
  p = counter_table.lookup(&uid);                            
  if (p != 0) {                                              
     counter = *p;
  }
  counter++;                                                 
  counter_table.update(&uid, &counter);                      
  return 0;
}
```
1. `BPF_HASH()` is a BCC macro that defines a hash table map.
2. `bpf_get_current_uid_gid()` is a helper function used to obtain the user ID that is running the process that triggered this kprobe event. The user ID is held in the lowest 32 bits of the 64-bit value that gets returned. (The top 32 bits hold the group ID, but that part is masked out.)
3. Look for an entry in the hash table with a key matching the user ID. It returns a pointer to the corresponding value in the hash table.
4. If there is an entry for this user ID, set the `counter` variable to the current value in the hash table (pointed to by `p`). If there is no entry for this user ID in the hash table, the pointer will be `0`, and the counter value will be left at `0`.
5. Whatever the current counter value is, it gets incremented by one.
6. Update the hash table with the new counter value for this user ID.
Take a closer look at the lines of code that access the hash table:
```c
  p = counter_table.lookup(&uid);
```

And later:
```c
  counter_table.update(&uid, &counter);

```

If you’re thinking “that’s not proper C code!” you’re absolutely right. C doesn’t support defining methods on structures like that.[^5] This is a great example where BCC’s version of C is very loosely a C-like language that BCC rewrites before it sends the code to the compiler. BCC offers some convenient shortcuts and macros that it converts into “proper” C.

Just like in the previous example, the C code is defined as a string called `program`. The program is compiled, loaded into the kernel, and attached to the `execve` kprobe, in exactly the same way as the previous “Hello World” example:

    b

This time a little more work is required on the Python side to read the information out of the hash table:

    while

[[#code_id_2_7|]]

This part of the code loops indefinitely, looking for output to display every two seconds.

[[#code_id_2_8|]]

BCC automatically creates a Python object to represent the hash table. This code loops through any values and prints them to the screen.

When you run this example, you’ll want a second terminal window where you can run some commands. Here’s some example output I obtained, annotated on the right side with the commands I ran in another terminal:

Terminal 1                          Terminal 2
$ ./hello-map.py 
                                    \[blank line(s) until I run something\]
ID 501: 1                           ls 
ID 501: 1
ID 501: 2                           ls
ID 501: 3       ID 0: 1             sudo ls
ID 501: 4       ID 0: 1             ls
ID 501: 4       ID 0: 1
ID 501: 5       ID 0: 2             sudo ls

This example generates a line of output every two seconds, whether anything has happened or not. At the end of this output, the hash table contains two entries:

*   `key=501, value=5`
    
*   `key=0, value=2`
    

In the second terminal, I have the user ID of 501. Running the `ls` command with this user ID increments the `execve` counter. When I run `sudo ls`, this results in two calls to `execve`: one is the execution of `sudo`, under user ID 501; the other is the execution of `ls`, under root’s user ID of 0.

In this example, I used a hash table to convey data from the eBPF program to user space. (I could also have used an array type of map here, since the key was an integer; hash tables let you use an arbitrary type as the key.) Hash tables are very convenient when the data is naturally in key–value pairs, but the user space code has to keep polling the table on a regular basis. The Linux kernel already supported the [perf subsystem](https://oreil.ly/nTvvH) for sending data from the kernel to user space, and eBPF includes support for using perf buffers and their successor, BPF ring buffers. Let’s take a look.[[null|]][[null|]]