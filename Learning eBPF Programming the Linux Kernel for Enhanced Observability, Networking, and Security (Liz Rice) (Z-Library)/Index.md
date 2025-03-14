 

# Index

### A

*   attachment
    *   of program to event, [[ch03.xhtml#ch03.xhtml15|Attaching to an Event]]\-[[ch03.xhtml#idm45305429895136|Attaching to an Event]]
    *   user space, [[ch07.xhtml#ch07.xhtml9|User Space Attachments]]\-[[ch07.xhtml#idm45305423912928|User Space Attachments]]
*   attachment types
    *   BPF, [[ch07.xhtml#idm45305423787936|BPF Attachment Types]]
    *   eBPF programs and, [[ch07.xhtml#ch07.xhtml0|eBPF Program and Attachment Types]]\-[[ch07.xhtml#idm45305423717664|Summary]]
*   Aya project, [[ch10.xhtml#ch10.xhtml14|Aya]]\-[[ch10.xhtml#idm45305419674800|Aya]]

### B

*   BCC framework
    *   BPF function calls, [[ch02.xhtml#idm45305431386176|Function Calls]]
    *   "Hello World" application, [[ch02.xhtml#ch02.xhtml1|eBPF’s “Hello World”]]\-[[ch02.xhtml#idm45305430538336|Summary]]
    *   portability approach, [[ch05.xhtml#idm45305428610896|BCC’s Approach to Portability]]
    *   Python/Lua/C++, [[ch10.xhtml#ch10.xhtml3|BCC Python/Lua/C++]]\-[[ch10.xhtml#idm45305420598448|BCC Python/Lua/C++]]
*   Berkeley Packet Filter (BPF)
    *   eBPF origins and, [[ch01.xhtml#idm45305443545856|eBPF’s Roots: The Berkeley Packet Filter]]
    *   evolution to eBPF, [[ch01.xhtml#idm45305440694112|From BPF to eBPF]]
*   Borkmann, Daniel, [[ch01.xhtml#idm45305440479328|The Evolution of eBPF to Production Systems]], [[ch11.xhtml#idm45305419616656|The Future Evolution of eBPF]]
*   BPF attachment types, [[ch07.xhtml#idm45305423786560|BPF Attachment Types]]
*   BPF trampoline, [[ch07.xhtml#idm45305424382688|Fentry/Fexit]]
*   BPF Type Format (BTF) (see BTF)
*   bpf() system calls, [[ch04.xhtml#ch04.xhtml0|The bpf() System Call]]\-[[ch04.xhtml#idm45305428651216|Summary]]
    *   attaching to kprobe events, [[ch04.xhtml#idm45305428868688|Attaching to Kprobe Events]]
    *   creating maps, [[ch04.xhtml#idm45305429074592|Creating Maps]]
    *   libppf wrapper around, [[ch05.xhtml#idm45305426782624|The Libbpf Library for User Space]]
    *   loading BTF data, [[ch04.xhtml#idm45305429127856|Loading BTF Data]]
    *   loading programs, [[ch04.xhtml#idm45305429056400|Loading a Program]]
    *   modifying a map from user space, [[ch04.xhtml#ch04.xhtml2|Modifying a Map from User Space]]\-[[ch04.xhtml#idm45305428935744|Modifying a Map from User Space]]
    *   perf buffers, [[ch04.xhtml#idm45305428885280|Initializing the Perf Buffer]]
    *   perf events, [[ch04.xhtml#idm45305428835344|Setting Up and Reading Perf Events]]
    *   program and map references, [[ch04.xhtml#ch04.xhtml4|BPF Program and Map References]]\-[[ch04.xhtml#idm45305428894064|BPF Links]]
    *   reading information from a map, [[ch04.xhtml#ch04.xhtml11|Reading Information from a Map]]\-[[ch04.xhtml#idm45305428668464|Reading Map Elements]]
    *   ring buffers, [[ch04.xhtml#ch04.xhtml9|Ring Buffers]]\-[[ch04.xhtml#idm45305428756048|Ring Buffers]]
    *   testing with BPF\_PROG\_RUN, [[ch10.xhtml#idm45305419662080|Testing BPF Programs]]
*   bpftool
    *   attaching program to event with, [[ch03.xhtml#idm45305429927264|Attaching to an Event]]
    *   auto-generating BPF skeleton code with, [[ch05.xhtml#ch05.xhtml17|BPF Skeletons]]\-[[ch05.xhtml#idm45305426068624|Managing an event buffer]]
    *   BPF relocations, [[ch05.xhtml#idm45305426821104|BPF Relocations]]
    *   detaching program from network interface with, [[ch03.xhtml#idm45305429855968|Detaching the Program]]
    *   generating dump of JIT-compiled code with, [[ch03.xhtml#idm45305429947072|The JIT-Compiled Machine Code]]
    *   and helper functions, [[ch07.xhtml#idm45305424660832|Helper Functions and Return Codes]]
    *   inspecting BTF types with, [[ch05.xhtml#idm45305428203264|Inspecting BTF Data for Maps and Programs]]
    *   kernel header file generation, [[ch05.xhtml#idm45305428156576|Generating a Kernel Header File]]
    *   listing BTF information with, [[ch05.xhtml#ch05.xhtml1|Listing BTF Information with bpftool]]\-[[ch05.xhtml#idm45305428526368|Listing BTF Information with bpftool]]
    *   listing programs with, [[ch03.xhtml#idm45305429999744|Inspecting the Loaded Program]]
    *   loading program into kernel with, [[ch03.xhtml#idm45305430025472|Loading the Program into the Kernel]], [[ch03.xhtml#idm45305429710144|BPF to BPF Calls]]
    *   perf subcommand, [[ch07.xhtml#idm45305424641216|Tracing]]
    *   pinning with, [[ch04.xhtml#idm45305428915936|Pinning]]
    *   reading map information, [[ch04.xhtml#readingMapInformation|Reading Information from a Map]]\-[[ch04.xhtml#idm45305428708080|Reading Map Elements]]
    *   removing program from kernel with, [[ch03.xhtml#idm45305429843792|Unloading the Program]]
    *   showing maps loaded into kernel, [[ch03.xhtml#idm45305429883568|Global Variables]]
    *   showing translated code with, [[ch03.xhtml#idm45305429961872|The Translated Bytecode]]
    *   skeleton generation, [[ch05.xhtml#idm45305428578960|CO-RE Overview]]
    *   viewing map contents with, [[ch04.xhtml#idm45305428954240|Modifying a Map from User Space]]
    *   visualizing control flow with, [[ch06.xhtml#idm45305425948320|Visualizing Control Flow]]
    *   XDP and, [[ch08.xhtml#idm45305422754848|Load Balancing and Forwarding]]
*   bpftrace, [[ch10.xhtml#ch10.xhtml1|Bpftrace]]\-[[ch10.xhtml#idm45305420764864|Bpftrace]]
*   BPF\_MAP\_CREATE, [[ch04.xhtml#idm45305429070224|Creating Maps]]
*   BPF\_MAP\_GET\_FD\_BY\_ID, [[ch04.xhtml#idm45305428723632|Finding a Map]]
*   BPF\_MAP\_GET\_NEXT\_ID, [[ch04.xhtml#idm45305428727872|Finding a Map]]
*   BPF\_MAP\_UPDATE\_ELEM, [[ch04.xhtml#idm45305429000736|Modifying a Map from User Space]], [[ch04.xhtml#idm45305428886384|Initializing the Perf Buffer]], [[ch04.xhtml#idm45305428836448|Setting Up and Reading Perf Events]]
*   BPF\_OBJ\_GET\_INFO\_BY\_FD, [[ch04.xhtml#idm45305428719856|Finding a Map]]
*   BPF\_PERF\_OUTPUT, [[ch02.xhtml#idm45305431916128|Perf and Ring Buffer Maps]], [[ch02.xhtml#idm45305431691296|Perf and Ring Buffer Maps]]
*   BPF\_PROG\_ATTACH, [[ch04.xhtml#idm45305428658400|Summary]]
*   BPF\_PROG\_LOAD, [[ch04.xhtml#idm45305428646528|Exercises]]
*   BPF\_PROG\_RUN, [[ch10.xhtml#idm45305419663184|Testing BPF Programs]]
*   BPF\_RAW\_TRACEPOINT\_OPEN, [[ch04.xhtml#idm45305428656656|Summary]]
*   BSD Packet Filter (see Berkeley Packet Filter)
*   BTF (BPF Type Format), [[ch05.xhtml#ch05.xhtml0|BPF Type Format]]\-[[ch05.xhtml#idm45305428170496|Inspecting BTF Data for Maps and Programs]]
    *   BTF types, [[ch05.xhtml#ch05.xhtml3|BTF Types]]\-[[ch05.xhtml#idm45305428421472|BTF Types]]
    *   data for functions/function prototypes, [[ch05.xhtml#idm45305428278528|BTF Data for Functions and Function Prototypes]]
    *   information in object file, [[ch05.xhtml#idm45305426946960|BTF Information in the Object File]]
    *   inspecting BTF data for maps and programs, [[ch05.xhtml#idm45305428201920|Inspecting BTF Data for Maps and Programs]]
    *   introduction of, [[ch01.xhtml#idm45305443678400|The Evolution of eBPF to Production Systems]]
    *   kernel headers, [[ch05.xhtml#ch05.xhtml4|Generating a Kernel Header File]]\-[[ch05.xhtml#idm45305428142560|Generating a Kernel Header File]]
    *   listing BTF information with bpftool, [[ch05.xhtml#ch05.xhtml2|Listing BTF Information with bpftool]]\-[[ch05.xhtml#idm45305428527744|Listing BTF Information with bpftool]]
    *   loading BTF data, [[ch04.xhtml#idm45305429126480|Loading BTF Data]]
    *   maps with BTF information, [[ch05.xhtml#idm45305428399872|Maps with BTF Information]]
    *   use cases, [[ch05.xhtml#idm45305428565792|BTF Use Cases]]
*   BTF-enabled tracepoints, [[ch07.xhtml#idm45305424009296|BTF-Enabled Tracepoints]]
*   bytecode
    *   JIT-compiled machine code, [[ch03.xhtml#idm45305429955648|The JIT-Compiled Machine Code]]
    *   translated, [[ch03.xhtml#idm45305429965552|The Translated Bytecode]]

### C

*   C (language)
    *   eBPF programming in, [[ch10.xhtml#ch10.xhtml6|C and Libbpf]]\-[[ch10.xhtml#idm45305420033056|Libbpfgo]]
    *   "Hello World" application, [[ch02.xhtml#ch02.xhtml2|eBPF’s “Hello World”]]\-[[ch02.xhtml#idm45305430539552|Summary]]
    *   kernel-side code in, [[ch10.xhtml#idm45305420755648|Language Choices for eBPF in the Kernel]]
*   C++, BCC tools in, [[ch10.xhtml#idm45305420607184|BCC Python/Lua/C++]]
*   cgroups (control groups), [[ch07.xhtml#idm45305423807232|Cgroups]]
*   Cilium
    *   coordinated network programs, [[ch08.xhtml#ch08.xhtml22|Coordinated Network Programs]]\-[[ch08.xhtml#idm45305421357280|Coordinated Network Programs]]
    *   ebpf-go library, [[ch10.xhtml#ch10.xhtml9|Ebpf-go]]\-[[ch10.xhtml#idm45305420175232|Ebpf-go]]
    *   origins, [[ch01.xhtml#idm45305440490400|The Evolution of eBPF to Production Systems]]
*   Cilium Tetragon, [[ch09.xhtml#ch09.xhtml13|Cilium Tetragon]]\-[[ch09.xhtml#idm45305420825696|Preventative Security]]
    *   attaching to internal kernel functions, [[ch09.xhtml#idm45305420846448|Attaching to Internal Kernel Functions]]
    *   preventative security, [[ch09.xhtml#idm45305420838368|Preventative Security]]
*   Clang compiler, [[ch03.xhtml#idm45305430161552|Compiling an eBPF Object File]], [[ch05.xhtml#idm45305427185072|Memory Access with CO-RE]]
*   cloud native environments, [[ch01.xhtml#ch01.xhtml7|eBPF in Cloud Native Environments]]\-[[ch01.xhtml#idm45305439787632|eBPF in Cloud Native Environments]]
*   CNI (Container Network Interface), [[ch08.xhtml#idm45305421382016|Avoiding iptables]], [[ch08.xhtml#idm45305421346160|Network Policy Enforcement]]
*   CO-RE (compile once, run everywhere) programs, [[ch05.xhtml#ch05.xhtml7|CO-RE eBPF Programs]]\-[[ch05.xhtml#idm45305427031152|License Definition]]
    *   basics, [[ch05.xhtml#idm45305428616576|CO-RE, BTF, and Libbpf]]
    *   BPF relocation information, [[ch05.xhtml#idm45305426932240|BPF Relocations]]
    *   BTF information in object file, [[ch05.xhtml#idm45305426943600|BTF Information in the Object File]]
    *   compiling eBPF programs for, [[ch05.xhtml#ch05.xhtml13|Compiling eBPF Programs for CO-RE]]\-[[ch05.xhtml#idm45305426936016|BTF Information in the Object File]]
    *   defining maps, [[ch05.xhtml#idm45305428059888|Defining Maps]]
    *   eBPF program sections, [[ch05.xhtml#ch05.xhtml10|eBPF Program Sections]]\-[[ch05.xhtml#idm45305427240384|eBPF Program Sections]]
    *   header files, [[ch05.xhtml#ch05.xhtml8|Header Files]]\-[[ch05.xhtml#idm45305428063296|Application-specific headers]]
    *   license definition, [[ch05.xhtml#idm45305427070272|License Definition]]
    *   memory access with, [[ch05.xhtml#ch05.xhtml11|Memory Access with CO-RE]]\-[[ch05.xhtml#idm45305427032656|License Definition]]
    *   overview, [[ch05.xhtml#idm45305428596272|CO-RE Overview]]
    *   user space code, [[ch05.xhtml#idm45305426815280|CO-RE User Space Code]]
*   CO-RE relocations, [[ch05.xhtml#idm45305428585504|CO-RE Overview]]
*   code examples, GitHub repository for, [[preface02.xhtml#idm45305443657104|Example Code and Exercises]]
*   compilation to eBPF byte code, [[ch05.xhtml#ch05.xhtml14|Compiling eBPF Programs for CO-RE]]\-[[ch05.xhtml#idm45305426937392|BTF Information in the Object File]]
    *   debug information, [[ch05.xhtml#idm45305426999152|Debug Information]]
    *   Makefile instruction, [[ch05.xhtml#idm45305426976848|Makefile]]
    *   optimization, [[ch05.xhtml#idm45305426993312|Optimization]]
    *   target architecture, [[ch05.xhtml#idm45305426987920|Target Architecture]]
*   compilation, defined, [[ch03.xhtml#idm45305430503344|The eBPF Virtual Machine]]
*   complexity limit, [[ch01.xhtml#idm45305443671872|The Evolution of eBPF to Production Systems]]
*   Container Network Interface (CNI), [[ch08.xhtml#idm45305421380896|Avoiding iptables]], [[ch08.xhtml#idm45305421347280|Network Policy Enforcement]]
*   containers, cgroups and, [[ch07.xhtml#idm45305423806016|Cgroups]]
*   context arguments, [[ch07.xhtml#idm45305424703824|Program Context Arguments]]
*   context information, accessing with verifier, [[ch06.xhtml#idm45305425166448|Accessing Context]]

### D

*   DDoS protection, [[ch08.xhtml#idm45305423638704|Packet Drops]]
*   debugging, [[ch05.xhtml#idm45305426997936|Debug Information]]
*   dereferencing pointers, [[ch06.xhtml#idm45305425352432|Checking Pointers Before Dereferencing Them]]
*   detaching program from network interface, [[ch03.xhtml#idm45305429859984|Detaching the Program]]
*   documentation, [[ch03.xhtml#idm45305430042784|Inspecting an eBPF Object File]]
*   drop counter, [[ch02.xhtml#idm45305431903056|Perf and Ring Buffer Maps]]
*   dynamic loading, [[ch01.xhtml#idm45305440517168|Dynamic Loading of eBPF Programs]]

### E

*   eBPF (generally)
    *   adding new functionality to kernel, [[ch01.xhtml#ch01.xhtml5|Adding New Functionality to the Kernel]]\-[[ch01.xhtml#idm45305444148848|Adding New Functionality to the Kernel]]
    *   basics, [[ch01.xhtml#ch01.xhtml0|What Is eBPF, and Why Is It Important?]]\-[[ch01.xhtml#idm45305439782880|Summary]]
    *   Berkeley Packet Filter and, [[ch01.xhtml#idm45305443544464|eBPF’s Roots: The Berkeley Packet Filter]]
    *   cloud native environments and, [[ch01.xhtml#ch01.xhtml8|eBPF in Cloud Native Environments]]\-[[ch01.xhtml#idm45305444174608|eBPF in Cloud Native Environments]]
    *   dynamic loading of eBPF programs, [[ch01.xhtml#idm45305440516064|Dynamic Loading of eBPF Programs]]
    *   evolution from BPF, [[ch01.xhtml#idm45305440692672|From BPF to eBPF]]
    *   evolution to production systems, [[ch01.xhtml#ch01.xhtml1|The Evolution of eBPF to Production Systems]]\-[[ch01.xhtml#idm45305443645056|The Evolution of eBPF to Production Systems]]
    *   helper functions, [[ch07.xhtml#idm45305424697280|Helper Functions and Return Codes]]
    *   high performance of eBPF programs, [[ch01.xhtml#idm45305440507184|High Performance of eBPF Programs]]
    *   kernel modules, [[ch01.xhtml#idm45305444143664|Kernel Modules]]
    *   Linux kernel and, [[ch01.xhtml#ch01.xhtml3|The Linux Kernel]]\-[[ch01.xhtml#idm45305444167056|The Linux Kernel]]
    *   programs (see programs, eBPF)
    *   terminology, [[ch01.xhtml#idm45305443641472|Naming Is Hard]]
    *   virtual machine (see virtual machine)
*   eBPF for Windows, [[ch11.xhtml#ch11.xhtml1|eBPF for Windows]]\-[[ch11.xhtml#idm45305419582976|eBPF for Windows]]
*   eBPF Foundation, [[ch11.xhtml#idm45305419611168|The eBPF Foundation]]
*   ebpf-go, [[ch10.xhtml#ch10.xhtml10|Ebpf-go]]\-[[ch10.xhtml#idm45305420176448|Ebpf-go]]
*   encapsulation, lightweight tunnels and, [[ch07.xhtml#idm45305423816480|Lightweight Tunnels]]
*   encryption
    *   encrypted connections in Kubernetes, [[ch08.xhtml#ch08.xhtml23|Encrypted Connections]]\-[[ch08.xhtml#idm45305421313136|Encrypted Connections]]
    *   packet encryption/decryption, [[ch08.xhtml#ch08.xhtml12|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421408896|User Space SSL Libraries]]
    *   transparent, [[ch08.xhtml#idm45305421330896|Encrypted Connections]]
*   event
    *   attaching eBPF to kprobe events, [[ch04.xhtml#idm45305428864288|Attaching to Kprobe Events]]
    *   attaching program to, [[ch03.xhtml#ch03.xhtml16|Attaching to an Event]]\-[[ch03.xhtml#idm45305429896512|Attaching to an Event]]
*   event buffer, [[ch05.xhtml#idm45305426209168|Managing an event buffer]]
*   execve syscall function, [[ch05.xhtml#idm45305426233104|Attaching to events]]

### F

*   Facebook, Katran and, [[ch01.xhtml#idm45305440487344|The Evolution of eBPF to Production Systems]]
*   Falco, [[ch09.xhtml#idm45305421034768|Syscall-Tracking Security Tools]]
*   fentry programs, [[ch07.xhtml#idm45305424381584|Fentry/Fexit]]
*   fexit programs, [[ch07.xhtml#idm45305424380480|Fentry/Fexit]]
*   file descriptor, [[ch04.xhtml#idm45305429079776|Loading BTF Data]]
*   firewalling
    *   defined, [[ch08.xhtml#idm45305423640784|Packet Drops]]
    *   network policy enforcement and, [[ch08.xhtml#idm45305421349216|Network Policy Enforcement]]
*   flow dissector, [[ch07.xhtml#idm45305423822560|Flow Dissector]]
*   frame pointer, [[ch06.xhtml#idm45305425791136|Helper Function Arguments]]
*   function calls, [[ch02.xhtml#idm45305431418912|Function Calls]], [[ch03.xhtml#ch03.xhtml21|BPF to BPF Calls]]\-[[ch03.xhtml#idm45305429665152|BPF to BPF Calls]]
*   functions and function prototypes, BTF data for, [[ch05.xhtml#idm45305428277184|BTF Data for Functions and Function Prototypes]]

### G

*   global variables, [[ch03.xhtml#ch03.xhtml18|Global Variables]]\-[[ch03.xhtml#idm45305429862800|Global Variables]]
*   Go, [[ch10.xhtml#idm45305420572416|Go]]
*   gobpf, [[ch10.xhtml#idm45305420566368|Gobpf]]
*   GPL license, [[ch06.xhtml#idm45305425785792|Checking the License]]
*   Gregg, Brendan, [[ch01.xhtml#idm45305440493136|The Evolution of eBPF to Production Systems]]

### H

*   hash table map, [[ch02.xhtml#ch02.xhtml8|Hash Table Map]]\-[[ch02.xhtml#idm45305431922240|Hash Table Map]]
*   header files, C, [[ch03.xhtml#idm45305430246544|eBPF “Hello World” for a Network Interface]]
    *   application-specific headers, [[ch05.xhtml#idm45305428091296|Application-specific headers]]
    *   for CO-RE eBPF programs, [[ch05.xhtml#ch05.xhtml9|Header Files]]\-[[ch05.xhtml#idm45305428064608|Application-specific headers]]
    *   generating a kernel header file, [[ch05.xhtml#ch05.xhtml5|Generating a Kernel Header File]]\-[[ch05.xhtml#idm45305428143936|Generating a Kernel Header File]]
    *   headers from libbpf, [[ch05.xhtml#idm45305428103456|Headers from libbpf]]
    *   kernel headers, [[ch05.xhtml#idm45305428114480|Kernel header information]]
*   helper functions, [[ch07.xhtml#idm45305424695904|Helper Functions and Return Codes]]

### I

*   infrared controllers, [[ch07.xhtml#idm45305423797648|Infrared Controllers]]
*   inspecting loaded programs, [[ch03.xhtml#ch03.xhtml12|Inspecting the Loaded Program]]\-[[ch03.xhtml#idm45305429936720|The JIT-Compiled Machine Code]]
    *   BPF program tag, [[ch03.xhtml#idm45305429977136|The BPF Program Tag]]
    *   JIT-compiled machine code, [[ch03.xhtml#idm45305429954208|The JIT-Compiled Machine Code]]
    *   translated bytecode, [[ch03.xhtml#idm45305429964176|The Translated Bytecode]]
*   instructions
    *   complexity limit, [[ch01.xhtml#idm45305443670800|The Evolution of eBPF to Production Systems]]
    *   for eBPF virtual machine, [[ch03.xhtml#ch03.xhtml4|eBPF Instructions]]\-[[ch03.xhtml#idm45305430349328|eBPF Instructions]]
    *   verifier check for invalid, [[ch06.xhtml#idm45305424948960|Invalid Instructions]]
    *   verifier check for unreachable, [[ch06.xhtml#idm45305424989776|Unreachable Instructions]]
*   ioctl, [[ch04.xhtml#ch04.xhtml8|Attaching to Kprobe Events]]\-[[ch04.xhtml#idm45305428816896|Setting Up and Reading Perf Events]]
*   Iovisor project, [[preface02.xhtml#idm45305444207664|Example Code and Exercises]]
*   IP addresses, Kubernetes and, [[ch08.xhtml#idm45305421350512|Network Policy Enforcement]]
*   ip link, [[ch03.xhtml#idm45305429915184|Attaching to an Event]], [[ch04.xhtml#idm45305428911856|Pinning]]
*   ip route, [[ch07.xhtml#idm45305423811136|Lightweight Tunnels]]
*   IPsec encryption protocol, [[ch08.xhtml#idm45305421327472|Encrypted Connections]]
*   iptables, avoiding, [[ch08.xhtml#idm45305421386016|Avoiding iptables]]

### J

*   Jacobson, Van, [[ch01.xhtml#idm45305443541168|eBPF’s Roots: The Berkeley Packet Filter]]
*   JIT compilation, [[ch03.xhtml#idm45305429952800|The JIT-Compiled Machine Code]]
*   jump instruction, [[ch02.xhtml#idm45305431364288|Function Calls]]

### K

*   Katran, [[ch01.xhtml#idm45305440486240|The Evolution of eBPF to Production Systems]]
*   kernel
    *   adding new functionality to, [[ch01.xhtml#ch01.xhtml6|Adding New Functionality to the Kernel]]\-[[ch01.xhtml#idm45305444150224|Adding New Functionality to the Kernel]]
    *   basics, [[ch01.xhtml#ch01.xhtml4|The Linux Kernel]]\-[[ch01.xhtml#idm45305444168432|The Linux Kernel]]
    *   defined, [[ch01.xhtml#idm45305442729664|The Linux Kernel]]
    *   evolution of features on, [[preface02.xhtml#idm45305444209184|Example Code and Exercises]]
    *   generating a kernel header file, [[ch05.xhtml#ch05.xhtml6|Generating a Kernel Header File]]\-[[ch05.xhtml#idm45305428145312|Generating a Kernel Header File]]
    *   inspecting loaded program in, [[ch03.xhtml#ch03.xhtml13|Inspecting the Loaded Program]]\-[[ch03.xhtml#idm45305429938096|The JIT-Compiled Machine Code]]
    *   loading eBPF program into, [[ch03.xhtml#idm45305430024032|Loading the Program into the Kernel]]
    *   removing program from, [[ch03.xhtml#idm45305429849024|Unloading the Program]]
*   kernel header file, [[ch05.xhtml#ch05.xhtml20|Generating a Kernel Header File]]\-[[ch05.xhtml#idm45305428146688|Generating a Kernel Header File]]
*   kernel modules, [[ch01.xhtml#idm45305444142288|Kernel Modules]]
*   kfuncs, [[ch07.xhtml#idm45305424654192|Kfuncs]]
*   kprobes, [[ch07.xhtml#ch07.xhtml4|Kprobes and Kretprobes]]\-[[ch07.xhtml#idm45305424386064|Attaching kprobes to other kernel functions]]
    *   attaching eBPF program to kprobe event, [[ch04.xhtml#idm45305428867072|Attaching to Kprobe Events]]
    *   attaching to syscall entry points, [[ch07.xhtml#idm45305424616704|Attaching kprobes to syscall entry points]]
    *   attaching to various kernel functions, [[ch07.xhtml#idm45305424588144|Attaching kprobes to other kernel functions]]
    *   origins, [[ch01.xhtml#idm45305440791632|The Evolution of eBPF to Production Systems]]
*   kube-proxy, [[ch08.xhtml#idm45305421384912|Avoiding iptables]]
*   Kubernetes
    *   avoiding iptables, [[ch08.xhtml#idm45305421383808|Avoiding iptables]]
    *   cgroups and, [[ch07.xhtml#idm45305423804912|Cgroups]]
    *   CNI, [[ch08.xhtml#idm45305421379776|Avoiding iptables]], [[ch08.xhtml#idm45305421345040|Network Policy Enforcement]]
    *   eBPF and Kubernetes networking, [[ch08.xhtml#ch08.xhtml19|eBPF and Kubernetes Networking]]\-[[ch08.xhtml#idm45305421310224|Encrypted Connections]]
    *   encrypted connections, [[ch08.xhtml#ch08.xhtml24|Encrypted Connections]]\-[[ch08.xhtml#idm45305421314512|Encrypted Connections]]
    *   policy enforcement, [[ch08.xhtml#idm45305421353680|Network Policy Enforcement]]
    *   sidecar model, [[ch01.xhtml#idm45305444190032|eBPF in Cloud Native Environments]]

### L

*   libbpf
    *   accessing maps, [[ch05.xhtml#idm45305426371824|Accessing existing maps]]
    *   BPF skeletons, [[ch05.xhtml#ch05.xhtml18|BPF Skeletons]]\-[[ch05.xhtml#idm45305426070000|Managing an event buffer]]
    *   code examples, [[ch05.xhtml#idm45305426065232|Libbpf Code Examples]]
    *   eBPF programming and, [[ch10.xhtml#ch10.xhtml7|C and Libbpf]]\-[[ch10.xhtml#idm45305420034272|Libbpfgo]]
    *   ebpf-go, [[ch10.xhtml#ch10.xhtml11|Ebpf-go]]\-[[ch10.xhtml#idm45305420238064|Ebpf-go]]
    *   header files for helper functions, [[ch05.xhtml#idm45305428102080|Headers from libbpf]]
    *   headers from, [[ch05.xhtml#idm45305428100688|Headers from libbpf]]
    *   loading programs/maps into kernel, [[ch05.xhtml#idm45305426475792|Loading programs and maps into the kernel]]
    *   user space, [[ch05.xhtml#ch05.xhtml15|The Libbpf Library for User Space]]\-[[ch05.xhtml#idm45305426036080|Libbpf Code Examples]]
*   libbpf-rs, [[ch10.xhtml#idm45305420022000|Libbpf-rs]]
*   libbpfgo, [[ch10.xhtml#idm45305420172880|Libbpfgo]]
*   licenses
    *   CO-RE and, [[ch05.xhtml#idm45305427066816|License Definition]]
    *   eBPF verifier and, [[ch06.xhtml#idm45305425756576|Checking the License]]
*   lightweight tunnels, [[ch07.xhtml#idm45305423815312|Lightweight Tunnels]]
*   links, BPF, [[ch04.xhtml#idm45305428900800|BPF Links]]
*   Linux kernel (see kernel)
*   LLVM, [[ch10.xhtml#idm45305420007520|Redbpf]]
*   llvm-objdump, [[ch03.xhtml#idm45305430096640|Inspecting an eBPF Object File]]
*   load balancing, [[ch08.xhtml#ch08.xhtml6|Load Balancing and Forwarding]]\-[[ch08.xhtml#idm45305422449568|Load Balancing and Forwarding]]
    *   kube-proxy, [[ch08.xhtml#idm45305421387392|Avoiding iptables]]
*   log, verifier, [[ch06.xhtml#ch06.xhtml2|The Verifier Log]]\-[[ch06.xhtml#idm45305425952944|The Verifier Log]]
*   loops, verifier handling of, [[ch06.xhtml#idm45305425131344|Loops]]
*   LSM (Linux Security Module)
    *   LSM BPF, [[ch01.xhtml#idm45305443675520|The Evolution of eBPF to Production Systems]], [[ch09.xhtml#ch09.xhtml11|BPF LSM]]\-[[ch09.xhtml#idm45305420916512|BPF LSM]]
    *   programs attached to, [[ch07.xhtml#idm45305423909712|LSM]]
*   Lua, BCC tools in, [[ch10.xhtml#idm45305420606080|BCC Python/Lua/C++]]

### M

*   Makefile, [[ch05.xhtml#idm45305426975456|Makefile]]
*   map references, [[ch04.xhtml#ch04.xhtml5|BPF Program and Map References]]\-[[ch04.xhtml#idm45305428895440|BPF Links]]
*   maps, BPF, [[ch02.xhtml#ch02.xhtml7|BPF Maps]]\-[[ch02.xhtml#idm45305430542048|Tail Calls]]
    *   accessing, [[ch05.xhtml#idm45305426370288|Accessing existing maps]]
    *   creating, [[ch04.xhtml#idm45305429073376|Creating Maps]]
    *   defined, [[ch02.xhtml#idm45305432393824|BPF Maps]]
    *   defining for CO-RE programs, [[ch05.xhtml#idm45305428058544|Defining Maps]]
    *   finding, [[ch04.xhtml#idm45305428741504|Finding a Map]]
    *   function calls and, [[ch02.xhtml#idm45305431417808|Function Calls]]
    *   hash table map, [[ch02.xhtml#ch02.xhtml9|Hash Table Map]]\-[[ch02.xhtml#idm45305431923456|Hash Table Map]]
    *   inspecting BTF data for, [[ch05.xhtml#idm45305428200576|Inspecting BTF Data for Maps and Programs]]
    *   maps with BTF information, [[ch05.xhtml#idm45305428398528|Maps with BTF Information]]
    *   modifying from user space, [[ch04.xhtml#ch04.xhtml3|Modifying a Map from User Space]]\-[[ch04.xhtml#idm45305428937088|Modifying a Map from User Space]]
    *   reading information from, [[ch04.xhtml#ch04.xhtml12|Reading Information from a Map]]\-[[ch04.xhtml#idm45305428669840|Reading Map Elements]]
    *   reading map elements, [[ch04.xhtml#idm45305428709552|Reading Map Elements]]
    *   repurposing semantics for use as global variables, [[ch03.xhtml#ch03.xhtml19|Global Variables]]\-[[ch03.xhtml#idm45305429864176|Global Variables]]
    *   tail calls and, [[ch02.xhtml#ch02.xhtml10|Tail Calls]]\-[[ch02.xhtml#idm45305430543392|Tail Calls]]
*   McCanne, Steven, [[ch01.xhtml#idm45305443540128|eBPF’s Roots: The Berkeley Packet Filter]]
*   memory access
    *   checks performed by verifier, [[ch06.xhtml#ch06.xhtml4|Checking Memory Access]]\-[[ch06.xhtml#idm45305425356032|Checking Memory Access]]
    *   with CO-RE, [[ch05.xhtml#ch05.xhtml12|Memory Access with CO-RE]]\-[[ch05.xhtml#idm45305427058144|License Definition]]
*   Meta, Katran and, [[ch01.xhtml#idm45305440488608|The Evolution of eBPF to Production Systems]]
*   Microsoft (eBPF for Windows), [[ch11.xhtml#ch11.xhtml3|eBPF for Windows]]\-[[ch11.xhtml#idm45305419584352|eBPF for Windows]]
*   modules, kernel, [[ch01.xhtml#idm45305440523936|Kernel Modules]]

### N

*   Nakryiko, Andrii, [[ch01.xhtml#idm45305443682896|The Evolution of eBPF to Production Systems]], [[ch11.xhtml#idm45305419615520|The Future Evolution of eBPF]]
*   network interface
    *   detaching program from, [[ch03.xhtml#idm45305429858784|Detaching the Program]]
    *   "Hello World" program for, [[ch03.xhtml#ch03.xhtml7|eBPF “Hello World” for a Network Interface]]\-[[ch03.xhtml#idm45305430167392|eBPF “Hello World” for a Network Interface]]
*   network security, [[ch09.xhtml#idm45305420822288|Network Security]]
    *   network security, [[ch08.xhtml#ch08.xhtml15b|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421414400|User Space SSL Libraries]]
    *   packet drops, [[ch08.xhtml#ch08.xhtml3a|Packet Drops]]\-[[ch08.xhtml#idm45305422930064|XDP Packet Parsing]]
    *   packet encryption/decryption, [[ch08.xhtml#ch08.xhtml15a|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421413024|User Space SSL Libraries]]
*   networking
    *   eBPF for, [[ch08.xhtml#ch08.xhtml0|eBPF for Networking]]\-[[ch08.xhtml#idm45305421307488|Summary]]
    *   eBPF program types, [[ch07.xhtml#ch07.xhtml12|Networking]]\-[[ch07.xhtml#idm45305423791024|Infrared Controllers]]
    *   Kubernetes and eBPF, [[ch08.xhtml#ch08.xhtml20|eBPF and Kubernetes Networking]]\-[[ch08.xhtml#idm45305421311632|Encrypted Connections]]
    *   load balancing and forwarding, [[ch08.xhtml#ch08.xhtml7|Load Balancing and Forwarding]]\-[[ch08.xhtml#idm45305422450944|Load Balancing and Forwarding]]
    *   network security, [[ch08.xhtml#ch08.xhtml1a|Packet Drops]]\-[[ch08.xhtml#idm45305422927632|XDP Packet Parsing]]
    *   packet drops, [[ch08.xhtml#ch08.xhtml1|Packet Drops]]\-[[ch08.xhtml#idm45305422926416|XDP Packet Parsing]]
    *   packet encryption/decryption, [[ch08.xhtml#ch08.xhtml13|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421410272|User Space SSL Libraries]]
    *   policy enforcement, [[ch08.xhtml#idm45305421352304|Network Policy Enforcement]]
    *   security, [[ch09.xhtml#idm45305420821184|Network Security]]
    *   traffic control, [[ch08.xhtml#ch08.xhtml9|Traffic Control (TC)]]\-[[ch08.xhtml#idm45305421838144|Traffic Control (TC)]]
    *   XDP offloading, [[ch08.xhtml#idm45305422402368|XDP Offloading]]
*   networking program types, [[ch07.xhtml#ch07.xhtml13|Networking]]\-[[ch07.xhtml#idm45305423792400|Infrared Controllers]]
    *   cgroups, [[ch07.xhtml#idm45305423803536|Cgroups]]
    *   flow dissector, [[ch07.xhtml#idm45305423821360|Flow Dissector]]
    *   infrared controllers, [[ch07.xhtml#idm45305423796448|Infrared Controllers]]
    *   lightweight tunnels, [[ch07.xhtml#idm45305423814208|Lightweight Tunnels]]
    *   sockets, [[ch07.xhtml#idm45305423855936|Sockets]]
    *   tracing-related types versus, [[ch07.xhtml#idm45305423861040|Networking]]
    *   traffic control, [[ch07.xhtml#idm45305423846928|Traffic Control]]
    *   XDP, [[ch07.xhtml#idm45305423839312|XDP]]

### O

*   object file
    *   compiling, [[ch03.xhtml#idm45305430164496|Compiling an eBPF Object File]]
    *   inspecting, [[ch03.xhtml#ch03.xhtml9|Inspecting an eBPF Object File]]\-[[ch03.xhtml#idm45305430028128|Inspecting an eBPF Object File]]
*   observability tools, security tools versus, [[ch09.xhtml#ch09.xhtml1|Security Observability Requires Policy and Context]]\-[[ch09.xhtml#idm45305421274528|Security Observability Requires Policy and Context]]
*   opensnoop, [[ch10.xhtml#idm45305419649712|Multiple eBPF Programs]]
*   optimization, [[ch05.xhtml#idm45305426991920|Optimization]]

### P

*   packet drops, [[ch08.xhtml#ch08.xhtml2|Packet Drops]]\-[[ch08.xhtml#idm45305422928848|XDP Packet Parsing]]
    *   XDP packet parsing, [[ch08.xhtml#ch08.xhtml4|XDP Packet Parsing]]\-[[ch08.xhtml#idm45305422932624|XDP Packet Parsing]]
    *   XDP program return codes, [[ch08.xhtml#idm45305423633536|XDP Program Return Codes]]
*   packet encryption/decryption, [[ch08.xhtml#ch08.xhtml14|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421411648|User Space SSL Libraries]]
*   packet processing, [[ch03.xhtml#idm45305430340768|eBPF “Hello World” for a Network Interface]]
*   packet-of-death vulnerability, [[ch08.xhtml#idm45305423637056|Packet Drops]]
*   perf buffers
    *   initializing, [[ch04.xhtml#idm45305428883904|Initializing the Perf Buffer]]
    *   managing, [[ch05.xhtml#idm45305426208064|Managing an event buffer]]
    *   ring buffers versus, [[ch02.xhtml#idm45305431913760|Perf and Ring Buffer Maps]], [[ch04.xhtml#idm45305428800992|Ring Buffers]]
    *   setting up/reading perf events, [[ch04.xhtml#idm45305428833968|Setting Up and Reading Perf Events]]
*   perf ring buffer, [[ch02.xhtml#idm45305431918080|Perf and Ring Buffer Maps]]
*   perf-related programs, [[ch07.xhtml#idm45305424644112|Tracing]]
*   Performance Measurement Unit (PMU), [[ch04.xhtml#idm45305428856832|Attaching to Kprobe Events]]
*   perf\_event\_open(), [[ch04.xhtml#idm45305428859216|Attaching to Kprobe Events]]
*   pinning, [[ch04.xhtml#idm45305428923296|Pinning]]
*   pods, defined, [[ch08.xhtml#idm45305421402128|eBPF and Kubernetes Networking]]
*   pointers, checking before dereferencing, [[ch06.xhtml#idm45305425351328|Checking Pointers Before Dereferencing Them]]
*   policies, security observability and, [[ch09.xhtml#idm45305421282432|Security Observability Requires Policy and Context]]
*   portability (see CO-RE)
*   preventative security, [[ch09.xhtml#idm45305420836992|Preventative Security]]
*   privileges, [[ch02.xhtml#idm45305443924064|Running “Hello World”]]
    *   escalation, [[ch09.xhtml#idm45305420843520|Attaching to Internal Kernel Functions]]
    *   for running eBPF programs, [[ch02.xhtml#idm45305442401504|Running “Hello World”]], [[ch03.xhtml#idm45305430015648|Loading the Program into the Kernel]]
*   production systems, evolution of eBPF to, [[ch01.xhtml#ch01.xhtml2|The Evolution of eBPF to Production Systems]]\-[[ch01.xhtml#idm45305443646496|The Evolution of eBPF to Production Systems]]
*   program sections, [[ch05.xhtml#ch05.xhtml21|eBPF Program Sections]]\-[[ch05.xhtml#idm45305427237632|eBPF Program Sections]]
*   program tag, [[ch03.xhtml#idm45305429975744|The BPF Program Tag]]
*   programming, eBPF, [[ch10.xhtml#ch10.xhtml0|eBPF Programming]]\-[[ch10.xhtml#idm45305419628656|Summary]]
    *   BCC Python/Lua/C++, [[ch10.xhtml#ch10.xhtml4|BCC Python/Lua/C++]]\-[[ch10.xhtml#idm45305420599824|BCC Python/Lua/C++]]
    *   bpftrace, [[ch10.xhtml#ch10.xhtml2|Bpftrace]]\-[[ch10.xhtml#idm45305420766240|Bpftrace]]
    *   C and libbpf, [[ch10.xhtml#ch10.xhtml8|C and Libbpf]]\-[[ch10.xhtml#idm45305420035488|Libbpfgo]]
    *   Go, [[ch10.xhtml#idm45305420571312|Go]]
    *   language choices for eBPF in the kernel, [[ch10.xhtml#idm45305420762240|Language Choices for eBPF in the Kernel]]
    *   multiple BPF programs, [[ch10.xhtml#idm45305419651376|Multiple eBPF Programs]]
    *   Rust, [[ch10.xhtml#ch10.xhtml12|Rust]]\-[[ch10.xhtml#idm45305419669104|Rust-bcc]]
    *   testing, [[ch10.xhtml#idm45305419665664|Testing BPF Programs]]
*   programs, eBPF, [[ch03.xhtml#ch03.xhtml0|Anatomy of an eBPF Program]]\-[[ch03.xhtml#idm45305429655024|Summary]]
    *   anatomy of, [[ch03.xhtml#ch03.xhtml1|Anatomy of an eBPF Program]]\-[[ch03.xhtml#idm45305429656400|Summary]]
    *   attaching to an event, [[ch03.xhtml#ch03.xhtml17|Attaching to an Event]]\-[[ch03.xhtml#idm45305429897888|Attaching to an Event]]
    *   attaching to kprobe events, [[ch04.xhtml#idm45305428865680|Attaching to Kprobe Events]]
    *   attachment types, [[ch07.xhtml#ch07.xhtml1|eBPF Program and Attachment Types]]\-[[ch07.xhtml#idm45305423719040|Summary]]
    *   BPF to BPF calls, [[ch03.xhtml#ch03.xhtml22|BPF to BPF Calls]]\-[[ch03.xhtml#idm45305429666528|BPF to BPF Calls]]
    *   compiling an eBPF object file, [[ch03.xhtml#idm45305430163120|Compiling an eBPF Object File]]
    *   context arguments, [[ch07.xhtml#idm45305424702720|Program Context Arguments]]
    *   detaching from network interface, [[ch03.xhtml#idm45305429857408|Detaching the Program]]
    *   dynamic loading, [[ch01.xhtml#idm45305440514624|Dynamic Loading of eBPF Programs]]
    *   eBPF virtual machine, [[ch03.xhtml#ch03.xhtml2|The eBPF Virtual Machine]]\-[[ch03.xhtml#idm45305430346416|eBPF Instructions]]
    *   ensuring that programs run to completion, [[ch06.xhtml#idm45305425161072|Running to Completion]]
    *   global variables, [[ch03.xhtml#ch03.xhtml20|Global Variables]]\-[[ch03.xhtml#idm45305429865552|Global Variables]]
    *   helper functions and return codes, [[ch07.xhtml#idm45305424667040|Helper Functions and Return Codes]]
    *   high performance of, [[ch01.xhtml#idm45305440505648|High Performance of eBPF Programs]]
    *   inspecting an eBPF object file, [[ch03.xhtml#ch03.xhtml10|Inspecting an eBPF Object File]]\-[[ch03.xhtml#idm45305430029504|Inspecting an eBPF Object File]]
    *   inspecting BTF data for, [[ch05.xhtml#idm45305428199200|Inspecting BTF Data for Maps and Programs]]
    *   inspecting loaded program in kernel, [[ch03.xhtml#ch03.xhtml14|Inspecting the Loaded Program]]\-[[ch03.xhtml#idm45305429939472|The JIT-Compiled Machine Code]]
    *   kfuncs and, [[ch07.xhtml#idm45305424653088|Kfuncs]]
    *   loading into kernel, [[ch03.xhtml#idm45305430022640|Loading the Program into the Kernel]]
    *   loading with bpf() syscall, [[ch04.xhtml#idm45305429054928|Loading a Program]]
    *   networking types, [[ch07.xhtml#ch07.xhtml14|Networking]]\-[[ch07.xhtml#idm45305423793776|Infrared Controllers]]
    *   removing from kernel, [[ch03.xhtml#idm45305429847648|Unloading the Program]]
    *   sections, [[ch05.xhtml#ch05.xhtml22|eBPF Program Sections]]\-[[ch05.xhtml#idm45305427239008|eBPF Program Sections]]
    *   tail calls, [[ch02.xhtml#ch02.xhtml12|Tail Calls]]\-[[ch02.xhtml#idm45305430545824|Tail Calls]]
    *   tracing, [[ch07.xhtml#ch07.xhtml2|Tracing]]\-[[ch07.xhtml#idm45305423878496|LSM]]
    *   unloading, [[ch03.xhtml#idm45305429846272|Unloading the Program]]
*   pseudo filesystem, [[ch04.xhtml#idm45305428918128|Pinning]]
*   Python BCC tools, [[ch10.xhtml#ch10.xhtml5|BCC Python/Lua/C++]]\-[[ch10.xhtml#idm45305420601232|BCC Python/Lua/C++]]

### R

*   Red Hat Enterprise Linux (RHEL), [[ch01.xhtml#idm45305444155168|Adding New Functionality to the Kernel]]
*   redbpf, [[ch10.xhtml#idm45305420012368|Redbpf]]
*   references, program and map, [[ch04.xhtml#ch04.xhtml6|BPF Program and Map References]]\-[[ch04.xhtml#idm45305428896848|BPF Links]]
    *   BPF links and, [[ch04.xhtml#idm45305428899696|BPF Links]]
    *   pinning and, [[ch04.xhtml#idm45305428922192|Pinning]]
*   registers, for eBPF virtual machine, [[ch03.xhtml#idm45305430499632|eBPF Registers]]
*   relocations, BPF, [[ch05.xhtml#idm45305426930928|BPF Relocations]]
*   return code
    *   checking with verifier, [[ch06.xhtml#idm45305425067664|Checking the Return Code]]
    *   program types and, [[ch07.xhtml#idm45305424665056|Helper Functions and Return Codes]]
    *   XDP program return codes, [[ch08.xhtml#idm45305423632096|XDP Program Return Codes]]
*   RHEL (Red Hat Enterprise Linux), [[ch01.xhtml#idm45305444154000|Adding New Functionality to the Kernel]]
*   ring buffer map, [[ch02.xhtml#idm45305431917104|Perf and Ring Buffer Maps]]
*   ring buffers, [[ch04.xhtml#ch04.xhtml10|Ring Buffers]]\-[[ch04.xhtml#idm45305428757504|Ring Buffers]]
    *   basics, [[ch02.xhtml#idm45305431907472|Perf and Ring Buffer Maps]]
    *   drop counter, [[ch02.xhtml#idm45305431904272|Perf and Ring Buffer Maps]]
    *   perf buffers versus, [[ch02.xhtml#idm45305431912544|Perf and Ring Buffer Maps]], [[ch04.xhtml#idm45305428799584|Ring Buffers]]
*   Rust, [[ch10.xhtml#ch10.xhtml13|Rust]]\-[[ch10.xhtml#idm45305419670512|Rust-bcc]]
    *   Aya and, [[ch10.xhtml#ch10.xhtml15|Aya]]\-[[ch10.xhtml#idm45305419676320|Aya]]
    *   libbpf-rs and, [[ch10.xhtml#idm45305420020896|Libbpf-rs]]
    *   redbpf and, [[ch10.xhtml#idm45305420011264|Redbpf]]

### S

*   SEC(), [[ch03.xhtml#idm45305430240640|eBPF “Hello World” for a Network Interface]], [[ch05.xhtml#idm45305427787760|eBPF Program Sections]]
*   seccomp
    *   basics, [[ch09.xhtml#idm45305421265296|Seccomp]]
    *   generating seccomp profiles, [[ch09.xhtml#ch09.xhtml5|Generating Seccomp Profiles]]\-[[ch09.xhtml#idm45305421039696|Generating Seccomp Profiles]]
    *   introduction of, [[ch01.xhtml#idm45305440698960|eBPF’s Roots: The Berkeley Packet Filter]]
*   security, [[ch09.xhtml#ch09.xhtml0|eBPF for Security]]\-[[ch09.xhtml#idm45305420812192|Summary]]
    *   Cilium Tetragon, [[ch09.xhtml#ch09.xhtml14|Cilium Tetragon]]\-[[ch09.xhtml#idm45305420827040|Preventative Security]]
    *   encrypted connections in Kubernetes, [[ch08.xhtml#ch08.xhtml25|Encrypted Connections]]\-[[ch08.xhtml#idm45305421315888|Encrypted Connections]]
    *   generating seccomp profiles, [[ch09.xhtml#ch09.xhtml6|Generating Seccomp Profiles]]\-[[ch09.xhtml#idm45305421040976|Generating Seccomp Profiles]]
    *   LSM BPF, [[ch09.xhtml#ch09.xhtml12|BPF LSM]]\-[[ch09.xhtml#idm45305420917888|BPF LSM]]
    *   network security, [[ch09.xhtml#idm45305420819808|Network Security]]
    *   observability tools versus security tools, [[ch09.xhtml#ch09.xhtml2|Security Observability Requires Policy and Context]]\-[[ch09.xhtml#idm45305421275904|Security Observability Requires Policy and Context]]
    *   packet drops, [[ch08.xhtml#ch08.xhtml3|Packet Drops]]\-[[ch08.xhtml#idm45305422931280|XDP Packet Parsing]]
    *   packet encryption/decryption, [[ch08.xhtml#ch08.xhtml15|Packet Encryption and Decryption]]\-[[ch08.xhtml#idm45305421415808|User Space SSL Libraries]]
    *   seccomp, [[ch09.xhtml#idm45305421263696|Seccomp]]
    *   using syscalls for security events, [[ch09.xhtml#ch09.xhtml3|Using System Calls for Security Events]]\-[[ch09.xhtml#idm45305420986912|Syscall-Tracking Security Tools]]
*   sidecar model, [[ch01.xhtml#ch01.xhtml9|eBPF in Cloud Native Environments]]\-[[ch01.xhtml#idm45305444176112|eBPF in Cloud Native Environments]]
*   skeleton code
    *   accessing maps, [[ch05.xhtml#idm45305426368912|Accessing existing maps]]
    *   attaching program to events, [[ch05.xhtml#idm45305426232000|Attaching to events]]
    *   auto-generation of, [[ch05.xhtml#idm45305428580464|CO-RE Overview]]
    *   bpftool for auto-generating, [[ch05.xhtml#ch05.xhtml19|BPF Skeletons]]\-[[ch05.xhtml#idm45305426071376|Managing an event buffer]]
    *   loading programs/maps into kernel, [[ch05.xhtml#idm45305426474512|Loading programs and maps into the kernel]]
    *   managing an event buffer, [[ch05.xhtml#idm45305426206688|Managing an event buffer]]
*   sockets, [[ch07.xhtml#idm45305423854544|Sockets]]
*   spin locks, [[ch05.xhtml#idm45305428560048|BTF Use Cases]]
*   SSL libraries, [[ch08.xhtml#ch08.xhtml17|User Space SSL Libraries]]\-[[ch08.xhtml#idm45305421417152|User Space SSL Libraries]]
*   Starovoitov, Alexei, [[ch01.xhtml#idm45305443681792|The Evolution of eBPF to Production Systems]], [[ch11.xhtml#idm45305419614416|The Future Evolution of eBPF]]
*   state pruning, [[ch06.xhtml#idm45305425993280|The Verification Process]]
*   subprograms, eBPF, [[ch02.xhtml#idm45305431384800|Function Calls]], [[ch03.xhtml#ch03.xhtml23|BPF to BPF Calls]]\-[[ch03.xhtml#idm45305429663776|BPF to BPF Calls]]
*   system calls
    *   attaching kprobes to entry points, [[ch07.xhtml#idm45305424615312|Attaching kprobes to syscall entry points]]
    *   bpf() (see bpf() system calls)
    *   generating seccomp profiles, [[ch09.xhtml#ch09.xhtml7|Generating Seccomp Profiles]]\-[[ch09.xhtml#idm45305421095744|Generating Seccomp Profiles]]
    *   Linux kernel and, [[ch01.xhtml#idm45305442725712|The Linux Kernel]]
    *   seccomp and, [[ch09.xhtml#idm45305421262320|Seccomp]]
    *   syscall-tracking security tools, [[ch09.xhtml#ch09.xhtml9|Syscall-Tracking Security Tools]]\-[[ch09.xhtml#idm45305420989856|Syscall-Tracking Security Tools]]
    *   using for security events, [[ch09.xhtml#ch09.xhtml4|Using System Calls for Security Events]]\-[[ch09.xhtml#idm45305420988288|Syscall-Tracking Security Tools]]

### T

*   tag, BPF program, [[ch03.xhtml#idm45305429974640|The BPF Program Tag]], [[ch03.xhtml#idm45305429921696|Attaching to an Event]]
*   tail calls, [[ch02.xhtml#ch02.xhtml11|Tail Calls]]\-[[ch02.xhtml#idm45305430544608|Tail Calls]]
*   target architecture, specifying for CO-RE programs, [[ch05.xhtml#idm45305426986352|Target Architecture]]
*   TC (see traffic control)
*   testing BPF programs, [[ch10.xhtml#idm45305419664288|Testing BPF Programs]]
*   Tetragon (see Cilium Tetragon)
*   Time Of Check to Time Of Use (TOCTOU), [[ch09.xhtml#ch09.xhtml10|Syscall-Tracking Security Tools]]\-[[ch09.xhtml#idm45305420991648|Syscall-Tracking Security Tools]]
*   tracepoints, [[ch07.xhtml#ch07.xhtml6|Tracepoints]]\-[[ch07.xhtml#idm45305423960080|BTF-Enabled Tracepoints]]
*   tracing
    *   bpf\_trace\_printk(), [[ch02.xhtml#idm45305440263152|BCC’s “Hello World”]], [[ch02.xhtml#idm45305432413792|Running “Hello World”]], [[ch03.xhtml#idm45305430176944|eBPF “Hello World” for a Network Interface]]
    *   eBPF programs, perf-related, [[ch07.xhtml#ch07.xhtml3|Tracing]]\-[[ch07.xhtml#idm45305423879872|LSM]]
    *   fentry/fexit, [[ch07.xhtml#idm45305424379376|Fentry/Fexit]]
    *   kprobes and kretprobes, [[ch07.xhtml#ch07.xhtml5|Kprobes and Kretprobes]]\-[[ch07.xhtml#idm45305424387376|Attaching kprobes to other kernel functions]]
    *   LSM, [[ch07.xhtml#idm45305423888064|LSM]]
    *   trace pipe, [[ch02.xhtml#idm45305432400224|Running “Hello World”]], [[ch02.xhtml#idm45305431428432|Perf and Ring Buffer Maps]], [[ch03.xhtml#idm45305429904480|Attaching to an Event]]
    *   tracepoints, [[ch07.xhtml#ch07.xhtml7|Tracepoints]]\-[[ch07.xhtml#idm45305423961360|BTF-Enabled Tracepoints]]
    *   user space attachments, [[ch07.xhtml#ch07.xhtml10|User Space Attachments]]\-[[ch07.xhtml#idm45305423914304|User Space Attachments]]
*   traffic control (TC), [[ch07.xhtml#idm45305423845488|Traffic Control]], [[ch08.xhtml#ch08.xhtml10|Traffic Control (TC)]]\-[[ch08.xhtml#idm45305421839360|Traffic Control (TC)]], [[ch08.xhtml#idm45305421364672|Coordinated Network Programs]]
*   transparent encryption, [[ch08.xhtml#idm45305421329520|Encrypted Connections]]
*   tunnels, lightweight, [[ch07.xhtml#idm45305423812816|Lightweight Tunnels]]

### U

*   unloading programs, [[ch03.xhtml#idm45305429844896|Unloading the Program]], [[ch04.xhtml#idm45305428927392|BPF Program and Map References]]
*   unreachable instructions, verifier check for, [[ch06.xhtml#idm45305424988384|Unreachable Instructions]]
*   user space
    *   CO-RE, [[ch05.xhtml#idm45305426813712|CO-RE User Space Code]]
    *   events, [[ch07.xhtml#ch07.xhtml11|User Space Attachments]]\-[[ch07.xhtml#idm45305423915712|User Space Attachments]]
    *   and kernel, [[ch01.xhtml#idm45305442728032|The Linux Kernel]]
    *   libbpf library for, [[ch05.xhtml#ch05.xhtml16|The Libbpf Library for User Space]]\-[[ch05.xhtml#idm45305426037488|Libbpf Code Examples]]
    *   SSL libraries, [[ch08.xhtml#ch08.xhtml18|User Space SSL Libraries]]\-[[ch08.xhtml#idm45305421470256|User Space SSL Libraries]]
*   user statically defined tracepoints (USDTs), [[ch07.xhtml#idm45305423951312|User Space Attachments]]

### V

*   verifier, [[ch06.xhtml#ch06.xhtml0|The eBPF Verifier]]\-[[ch06.xhtml#idm45305424940752|Summary]]
    *   accessing context, [[ch06.xhtml#idm45305425165376|Accessing Context]]
    *   checking for invalid instructions, [[ch06.xhtml#idm45305424994224|Invalid Instructions]]
    *   checking for unreachable instructions, [[ch06.xhtml#idm45305424987312|Unreachable Instructions]]
    *   checking memory access, [[ch06.xhtml#ch06.xhtml5|Checking Memory Access]]\-[[ch06.xhtml#idm45305425357376|Checking Memory Access]]
    *   checking pointers before dereferencing them, [[ch06.xhtml#idm45305425350256|Checking Pointers Before Dereferencing Them]]
    *   checking return code, [[ch06.xhtml#idm45305425066288|Checking the Return Code]]
    *   checking the license, [[ch06.xhtml#idm45305425755360|Checking the License]]
    *   ensuring that programs run to completion, [[ch06.xhtml#idm45305425135840|Running to Completion]]
    *   helper function arguments, [[ch06.xhtml#idm45305425928112|Helper Function Arguments]]
    *   loops, [[ch06.xhtml#idm45305425130272|Loops]]
    *   validating helper functions, [[ch06.xhtml#idm45305425937856|Validating Helper Functions]]
    *   verification process, [[ch06.xhtml#ch06.xhtml1|The Verification Process]]\-[[ch06.xhtml#idm45305425988720|The Verification Process]]
    *   verifier log, [[ch06.xhtml#ch06.xhtml3|The Verifier Log]]\-[[ch06.xhtml#idm45305425954320|The Verifier Log]]
    *   visualizing control flow, [[ch06.xhtml#idm45305425949888|Visualizing Control Flow]]
*   virtual machine, eBPF, [[ch03.xhtml#ch03.xhtml3|The eBPF Virtual Machine]]\-[[ch03.xhtml#idm45305430347824|eBPF Instructions]]
    *   instructions, [[ch03.xhtml#ch03.xhtml5|eBPF Instructions]]\-[[ch03.xhtml#idm45305430350672|eBPF Instructions]]
    *   registers, [[ch03.xhtml#idm45305430498560|eBPF Registers]]

### W

*   wide instruction encoding, [[ch03.xhtml#idm45305430365984|eBPF Instructions]], [[ch03.xhtml#idm45305430045744|Inspecting an eBPF Object File]]
*   Windows, eBPF for, [[ch11.xhtml#ch11.xhtml4|eBPF for Windows]]\-[[ch11.xhtml#idm45305419585728|eBPF for Windows]]
*   WireGuard encryption protocol, [[ch08.xhtml#idm45305421326352|Encrypted Connections]]

### X

*   XDP (eXpress Data Path)
    *   load balancing and forwarding, [[ch08.xhtml#ch08.xhtml8|Load Balancing and Forwarding]]\-[[ch08.xhtml#idm45305422452288|Load Balancing and Forwarding]]
    *   memory access, [[ch06.xhtml#idm45305425741808|Checking Memory Access]]
    *   offloading, [[ch03.xhtml#idm45305430170672|eBPF “Hello World” for a Network Interface]], [[ch08.xhtml#idm45305422400992|XDP Offloading]]
    *   packet parsing, [[ch08.xhtml#ch08.xhtml5|XDP Packet Parsing]]\-[[ch08.xhtml#idm45305422933840|XDP Packet Parsing]]
    *   program types, [[ch07.xhtml#idm45305423837920|XDP]]
    *   return codes, [[ch07.xhtml#idm45305424663712|Helper Functions and Return Codes]], [[ch08.xhtml#idm45305423630704|XDP Program Return Codes]]

 

# About the Author

**Liz Rice** is the chief open source officer with eBPF specialists at Isovalent, creators of the Cilium cloud native networking, security, and observability project. She sits on the CNCF Governing Board and on the Board of OpenUK. She was chair of the CNCF’s Technical Oversight Committee in 2019–2022 and cochair of KubeCon + CloudNativeCon in 2018. She is also the author of _Container Security_ published by O’Reilly.

She has a wealth of software development, team, and product management experience from working on network protocols and distributed systems and in digital technology sectors such as VOD, music, and VoIP. When not writing code, or talking about it, Liz loves riding bikes in places with better weather than her native London, competing in virtual races on Zwift, and making music under the pseudonym Insider Nine.

# Colophon

The animal on the cover of _Learning eBPF_ is an early bumblebee (_Bombus pratorum_), a bumblebee species found in most of Europe (especially the UK) and parts of Asia.

_Bombus pratorum_ builds nests above ground in fields, parks, and sparse forest, even repurposing abandoned bird or rodent nests. The early bumblebee indeed emerges early in the year, typically from March to July, but in southern England, worker bees appear as early as February, making it quite common for two colony cycles to occur in a single year.

This species of bumblebee is quite smaller than others. While there are slight variations among queen, worker, and male drone types, the early bumblebee’s appearance is generally black with a yellow collar, another yellow band on the abdominal segment, and a reddish or dull orange tail.

Early bumblebees form colonies with queen and worker bees but, unusually, queens use aggressive behavior rather than pheromones to establish dominance, using her mandibles to head-butt the strongest worker bees to maintain control of the colony. Worker bees forages for nectar and pollen of flowering plants such as white clover, thistle, sage, lavender; drones are created late in the hive cycle and leave the nest in search of new queens.

Many of the animals on O’Reilly covers are endangered; all of them are important to the world. The cover illustration is by Karen Montgomery, based on an antique line engraving from _The Animal Kingdom Illustrated_. The cover fonts are Gilroy Semibold and Guardian Sans. The text font is Adobe Minion Pro; the heading font is Adobe Myriad Condensed; and the code font is Dalton Maag’s Ubuntu Mono.