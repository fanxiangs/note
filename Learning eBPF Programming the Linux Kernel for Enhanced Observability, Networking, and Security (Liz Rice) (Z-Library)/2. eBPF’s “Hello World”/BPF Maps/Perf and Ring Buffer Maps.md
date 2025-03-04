[[null|]][[null|]][[null|]]In this section I’m going to describe a slightly more sophisticated version of “Hello World” that uses BCC’s `BPF_PERF_OUTPUT` capabilities, which let you write data in a structure of your choosing into a perf ring buffer map.

>[!note]
>There is a newer construct called “BPF ring buffers” that are now generally preferred over BPF perf buffers, if you have a kernel of version 5.8 or above. Andrii Nakryiko discusses the difference in his [BPF ring buffer](https://oreil.ly/ARRyV) blog post. You’ll see an example of BCC’s `BPF_RINGBUF_OUTPUT` in [[ch04.xhtml#the_bpfleft_parenthesisright_parenthesi|Chapter 4]].
# Ring Buffers

Ring buffers are by no means unique to eBPF, but I’ll explain them just in case you haven’t come across them before. You can think of a ring buffer as a piece of memory logically organized in a ring, with separate “write” and “read” pointers. Data of some arbitrary length gets written to wherever the write pointer is, with the length information included in a header for that data. The write pointer moves to after the end of that data, ready for the next write operation.^ddd

Similarly, for a read operation, data gets read from wherever the read pointer is, using the header to determine how much data to read. The read pointer moves along in the same direction as the write pointer so that it points to the next available piece of data. This is illustrated in [[#a_ring_buffer|Figure 2-3]], showing a ring buffer with three items of different length available for reading.

If the read pointer catches up with the write pointer, it simply means there’s no data to read. [[null|]][[null|]]If a write operation would make the write pointer overtake the read pointer, the data doesn’t get written and a _drop counter_ gets incremented. Read operations include the drop counter to indicate whether data has been lost since the last successful read.

If read and write operations happened at precisely the same rate with no variability, and they always contained the same amount of data, you could at least in theory get away with a ring buffer just big enough to accommodate that data size. In most applications there will be some variation in the time between reads, writes, or both, so the buffer size needs to be tuned to account for this.

![A ring buffer](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0203.png)

###### Figure 2-3. A ring buffer

You’ll find the source code for this example in _chapter2/hello-buffer.py_ in the _Learning eBPF_ [GitHub repository](http://github.com/lizrice/learning-ebpf). As in the first “Hello World” example you saw early in this chapter, this version will write the string `"Hello World"` to the screen every time the `execve()` syscall is used. It will also look up the process ID and the name of the command that makes each `execve()` call so that you’ll get similar output to the first example. This gives me the opportunity to show you a couple more examples of BPF helper functions.

Here’s the eBPF program that will be loaded into the kernel:
```c
BPF_PERF_OUTPUT(output);                                                

struct data_t {                                                         
   int pid;
   int uid;
   char command[16];
   char message[12];
};

int hello(void *ctx) {
   struct data_t data = {};                                             
   char message[12] = "Hello World";

   data.pid = bpf_get_current_pid_tgid() >> 32;                         
   data.uid = bpf_get_current_uid_gid() & 0xFFFFFFFF;                   

   bpf_get_current_comm(&data.command, sizeof(data.command));            
   bpf_probe_read_kernel(&data.message, sizeof(data.message), message); 

   output.perf_submit(ctx, &data, sizeof(data));                        

   return 0;
}
```

1. [[null|]]BCC defines the macro `BPF_PERF_OUTPUT` for creating a map that will be used to pass messages from the kernel to user space. I’ve called this map `output`.
2. Every time `hello()` is run, the code will write a structure’s worth of data. This is the definition of that structure, which has fields for the process ID, the name of the currently running command, and a text message.
3. `data` is a local variable that holds the data structure to be submitted, and `message` holds the `"Hello World"` string.
4. `bpf_get_current_pid_tgid()` is a helper function that gets the ID of the process that triggered this eBPF program to run. It returns a 64-bit value with the process ID in the top 32 bits.[^6]
5. `bpf_get_current_uid_gid()` is the helper function you saw in the previous example for obtaining the user ID.
6. Similarly, `bpf_get_current_comm()` is a helper function for getting the name of the executable (or “command”) that’s running in the process that made the `execve` syscall. This is a string, not a numeric value like the process and user IDs, and in C you can’t simply assign a string using `=`. You have to pass the address of the field where the string should be written, `&data.command`, as an argument to the helper function.
7. For this example, the message is `"Hello World"` every time. `bpf_probe_read_kernel()` copies it into the right place in the data structure.
8. At this point the data structure is populated with the process ID, command name, and message. This call to `output.perf_submit()` puts that data into the map.

Just as in the first “Hello World” example, this C program is assigned to a string called `program` in the Python code. What follows is the rest of the Python code:

```c
b = BPF(text=program)                                
syscall = b.get_syscall_fnname("execve")
b.attach_kprobe(event=syscall, fn_name="hello")

def print_event(cpu, data, size):                    
   data = b["output"].event(data)
   print(f"{data.pid} {data.uid} {data.command.decode()} " + \
         f"{data.message.decode()}")

b["output"].open_perf_buffer(print_event)            
while True:                                          
   b.perf_buffer_poll()
```
1. The lines that compile the C code, load it into the kernel, and attach it to the syscall event are unchanged from the version of “Hello World” you saw earlier.
2. `print_event` is a callback function that will output a line of data to the screen. BCC does some heavy lifting so that I can refer to the map simply as `b["output"]` and grab data from it using `b["output"].event()`.
3. `b["output"].open_perf_buffer()` opens the perf ring buffer. The function takes `print_event` as an argument to define that this is the callback function to be used whenever there is data to read from the buffer.
4. The program will now loop indefinitely,[^7] polling the perf ring buffer. If there is any data available, `print_event` will get called.

Running this code gives us output that’s fairly similar to the original “Hello World”:

$ sudo ./hello-buffer.py
11654 node Hello World
11655 sh Hello World
...

As before, you might need to open a second terminal to the same (virtual) machine and run some commands to trigger some output.

[[null|]]The big difference between this and the original “Hello World” example is that instead of using a single, central trace pipe, the data is now being passed via a ring buffer map called `output` that was created by this program for its own use, as shown in [[#using_a_perf_ring_buffer_for_passing_da|Figure 2-4]].

![Using a perf ring buffer for passing data from the kernel to user space](/Learning%20eBPF%20Programming%20the%20Linux%20Kernel%20for%20Enhanced%20Observability,%20Networking,%20and%20Security%20(Liz%20Rice)%20(Z-Library)/images/lebp_0204.png)

###### Figure 2-4. Using a perf ring buffer for passing data from the kernel to user space

You can verify that the information isn’t going to the trace pipe by using `cat /sys/kernel/debug/tracing/trace_pipe`.

As well as demonstrating the use of a ring buffer map, this example shows some eBPF helper functions for retrieving contextual information about the event that triggered the eBPF program to run. Here you’ve seen helper functions getting the user ID, the process ID, and the name of the current command. As you’ll see in [[ch07.xhtml#ebpf_program_and_attachment_types|Chapter 7]], the set of contextual information that’s available and the set of valid helper functions that can be used to retrieve it depend on what type of program it is and what event triggered it.

The fact that contextual information like this is available to the eBPF code is what makes it so valuable for observability. Whenever an event occurs, an eBPF program can report not only the fact that the event happened but also relevant information about what happened to trigger the event. It’s also highly performant, since all this information can be gathered within the kernel, without the need for any synchronous context switching to user space.

You’ll see further examples in this book where eBPF helper functions are used to gather other contextual data, as well as examples where eBPF programs change the contextual data or even block events from happening altogether.