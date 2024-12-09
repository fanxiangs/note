Linux 系统是一个基于虚拟内存的系统，每个进程都有自己的虚拟地址空间，物理内存的映射是在需要时按需分配的。这种设计允许物理内存的过度订阅，Linux 通过页面置换守护进程和物理交换设备来管理，并且在最后的手段下会使用 OOM（Out-Of-Memory）杀手。Linux 会利用空闲内存作为文件系统缓存，这在下一章中会详细讨论。

本章介绍了如何使用 BPF 工具以新的方式展示应用程序内存使用情况，帮助你检查内核如何响应内存压力。随着 CPU 性能的提升，内存 I/O 成为了新的瓶颈，理解内存的使用方式可以帮助你找到很多性能提升的机会。

 **学习目标**
1. **理解内存分配和分页行为**  
    掌握 Linux 内存管理的基础，了解虚拟内存和物理内存的映射，以及分页的工作原理。
2. **学习成功分析内存行为的策略**  
    了解如何利用跟踪器进行内存行为分析，包括如何通过 BPF 和传统工具来检查内存使用情况。
3. **使用传统工具了解内存容量使用情况**  
    传统工具如 `free`, `vmstat`, `top`, `sar` 等可以帮助你查看系统的内存使用状态，包括空闲内存、缓存、交换空间等。
4. **使用 BPF 工具识别导致堆和 RSS（常驻集大小）增长的代码路径**  
    通过 BPF 工具，定位内存增长的来源，例如哪些函数或代码路径导致堆或进程的常驻集增大。
5. **按文件名和栈跟踪表征页面故障**  
    使用 BPF 工具跟踪页面错误，并关联到具体的文件和调用栈，这对于内存性能问题的分析至关重要。
6. **分析 VM 扫描器的行为**  
    了解 Linux 的虚拟内存管理（VM）扫描器如何工作，如何管理内存的回收和页面置换。
7. **确定内存回收的性能影响**  
    通过 BPF 工具监控内存回收过程，分析内存回收对系统性能的影响，识别何时内存回收成为性能瓶颈。
8. **识别哪些进程正在等待交换（swap）加载**  
    使用 BPF 工具找出哪些进程在等待交换回内存，识别因内存交换导致的性能问题。
9. **使用 bpftrace 一行命令以自定义方式探查内存使用**  
    学习如何通过 `bpftrace` 一行命令来对内存使用进行快速分析。
## **内存分析背景**
内存管理是 Linux 系统的一个核心部分，理解虚拟内存的分配和分页机制是性能调优的关键。以下是一些内存管理的基础概念：
- **虚拟内存**：每个进程有独立的虚拟地址空间，通过分页机制映射到物理内存。
- **物理内存和交换**：当物理内存不足时，Linux 会使用交换设备（swap）将不常用的页面移到磁盘，释放内存给更重要的任务。
- **OOM 杀手**：当系统内存不足时，OOM 杀手会终止一些进程来释放内存。
- **页面故障**：当进程访问的虚拟地址没有映射到物理内存时，会发生页面故障，导致内核加载对应的页面。
### **BPF 工具在内存分析中的应用**
BPF 在内存分析中可以帮助解决以下问题：
1. **分析内存分配和释放**：通过 BPF 跟踪内存分配函数（如 `malloc`）的调用，识别哪些代码路径导致了内存的增长。
2. **跟踪页面故障**：BPF 可以用来跟踪页面故障，分析哪些文件或函数引发了内存的页面缺页。
3. **监控内存回收**：通过跟踪内存回收的过程，了解系统在压力下的表现，识别可能导致性能下降的内存回收行为。
4. **识别等待交换的进程**：BPF 可以帮助你识别系统中哪些进程正在等待内存从交换区加载到物理内存，帮助你发现交换相关的性能问题。
### **传统内存分析工具**
在 BPF 工具之前，传统的内存分析工具已经可以帮助我们理解系统的内存使用情况，主要包括：
- **`free`**：查看系统的总内存、空闲内存、缓存和交换空间的使用情况。
- **`vmstat`**：显示内存、交换区、进程和 I/O 的统计信息。
- **`top`**：实时显示进程的内存和 CPU 使用情况。
- **`sar`**：用于收集、报告和保存系统的性能数据。
- **`pmap`**：查看进程的内存映射。

## TRADITIONAL TOOLS


## BPF TOOLS

本节介绍了用于内存性能分析和排查的 BPF 工具，如图 7-4 所示。这些工具来自 BCC 和 bpftrace 仓库（分别在第 4 章和第 5 章中介绍过），或者为本书专门创建。一些工具在 BCC 和 bpftrace 中都存在。以下是内存相关工具的分类及其功能概要。

### **工具来源**

- **BCC 工具**  
    大部分以 C/Python 实现，功能丰富，支持复杂的内存分析场景。
- **bpftrace 工具**  
    灵活轻便，适合快速编写和运行内存分析的单行脚本。

详细功能和选项请参考各工具的仓库文档。以下是重要内存工具的能力摘要，而关于内核内存分析的工具（如 `kmem`、`kpages`、`slabratetop`、`numamove` 等）将在第 14 章中介绍。

---

### **内存相关工具选摘**

|**工具名**|**描述**|**来源**|
|---|---|---|
|`memleak`|检测动态内存分配中的内存泄漏，适用于排查 C 程序中未释放的堆分配。|BCC|
|`allocsnoop`|跟踪内存分配调用，例如 `kmalloc` 和 `vmalloc`，并报告大小和调用栈。|本书工具|
|`stackcount`|收集内核或用户栈的调用频率，用于分析特定函数或代码路径的内存分配次数。|BCC/bpftrace|
|`mallocstacks`|跟踪用户空间程序的堆分配 (`malloc`)，并提供调用栈。|bpftrace|
|`pagefaultsnoop`|跟踪页面故障事件，包括缺页加载的文件和调用栈。|本书工具|
|`reclaimsnoop`|监控内核的内存回收行为，例如页面的回收和释放。|本书工具|
|`rssstat`|跟踪进程常驻集大小 (RSS) 的变化，并按进程或线程进行统计。|本书工具|
|`swapin`|跟踪哪些进程因内存交换而等待，帮助识别过多交换操作引发的性能问题。|本书工具|
|`vmscan`|分析 VM 扫描器的行为，包括页面置换和回收的频率。|本书工具|
|`biolatency`|监控磁盘 I/O 延迟，协助分析内存与 I/O 的交互关系，识别内存压力下的性能瓶颈。|BCC|

---

### **使用场景示例**

1. **检测内存泄漏**  
    使用 `memleak` 工具检测程序中的堆内存泄漏，报告未释放的分配调用。
    
    ```bash
    sudo memleak -a
    ```
    
2. **监控页面故障**  
    使用 `pagefaultsnoop` 工具跟踪页面故障，确定缺页来源和调用栈。
    
    ```bash
    sudo pagefaultsnoop
    ```
3. **分析内存回收**  
    使用 `reclaimsnoop` 工具监控内核的内存回收行为，评估内存压力的影响。
    
    ```bash
    sudo reclaimsnoop
    ```
    
4. **跟踪 RSS 增长**  
    使用 `rssstat` 工具按进程统计 RSS 的增长和变化，帮助定位内存使用异常的进程。
    
    ```bash
    sudo rssstat
    ```
    
5. **探查交换操作**  
    使用 `swapin` 工具分析哪些进程由于内存交换而受影响，协助优化内存分配。
    
    ```bash
    sudo swapin
    ```

---

### **总结**

BPF 工具为内存性能分析提供了细粒度的视角，通过跟踪内存分配、页面故障、内存回收和交换操作，可以精确定位性能问题的来源。在传统工具提供宏观视图的基础上，BPF 工具能够补充更多细节分析。结合使用这些工具，能够更全面地优化系统的内存使用和性能表现。

## BPF ONE-LINERS
这一部分展示了如何使用 **BCC** 和 **bpftrace** 来实现简洁的 BPF（Berkeley Packet Filter）监控任务。这两种工具分别以不同的方式提供了对系统行为的实时观测，但都基于 Linux 内核的 BPF 技术。

### **BCC**

BCC 提供了一组专用工具，如 `stackcount`、`trace` 和 `funccount`，用于捕获和分析特定事件。

1. **统计进程堆扩展 (brk()) 的用户态调用栈**
    ```bash
    stackcount -U t:syscalls:sys_enter_brk
    ```
    
    解释：监控系统调用 `brk()` 的执行路径，并基于用户态调用栈进行统计。
    
2. **统计用户页错误的用户态调用栈**
    ```bash
    stackcount -U t:exceptions:page_fault_user
    ```
    
    解释：跟踪用户页错误的发生情况，通过用户态栈识别潜在的代码路径。
    
3. **统计 vmscan 操作**
    
    ```bash
    funccount 't:vmscan:*'
    ```
    
    解释：监控所有与内存扫描相关的内核 tracepoint。
    
4. **展示使用 hugepages 的 madvise() 调用**
    
    ```bash
    trace hugepage_madvise
    ```
    
    解释：记录哪些进程调用了 `madvise()` 函数来请求 hugepages。
    
5. **统计页迁移次数**
    
    ```bash
    funccount t:migrate:mm_migrate_pages
    ```
    
    解释：追踪与页迁移相关的内核事件。
    
6. **跟踪内存压缩事件**
    
    ```bash
    trace t:compaction:mm_compaction_begin
    ```
    
    解释：记录内存压缩开始时的事件。
    

---

### **bpftrace**

**bpftrace** 提供了一种简单的脚本语言，支持更动态、更灵活的实时监控。

1. **统计进程堆扩展 (brk()) 的代码路径**
    
    ```bash
    bpftrace -e 'tracepoint:syscalls:sys_enter_brk { @[ustack, comm] = count(); }'
    ```
    
    解释：基于调用栈和进程名（`comm`）统计 `brk()` 的触发频率。
    
2. **按进程统计页错误**
    
    ```bash
    bpftrace -e 'software:page-fault:1 { @[comm, pid] = count(); }'
    ```
    
    解释：监控页错误（通过软件异常）并统计每个进程的频率。
    
3. **统计用户页错误的用户态调用栈**
    
    ```bash
    bpftrace -e 'tracepoint:exceptions:page_fault_user { @[ustack, comm] = count(); }'
    ```
    
    解释：分析哪些用户态代码路径导致了页错误。
    
4. **统计 vmscan 操作**
    
    ```bash
    bpftrace -e 'tracepoint:vmscan:* { @[probe] = count(); }'
    ```
    
    解释：统计所有与内存扫描相关的 tracepoint，并显示其触发次数。
    
5. **展示使用 hugepages 的 madvise() 调用**
    
    ```bash
    bpftrace -e 'kprobe:hugepage_madvise { printf("%s by PID %d\n", probe, pid); }'
    ```
    
    解释：记录 `madvise()` 请求 hugepages 的内核函数调用及其进程 ID。
    
6. **统计页迁移次数**
    
    ```bash
    bpftrace -e 'tracepoint:migrate:mm_migrate_pages { @ = count(); }'
    ```
    
    解释：监控页面迁移相关的内核 tracepoint。
    
7. **跟踪内存压缩事件**
    
    ```bash
    bpftrace -e 't:compaction:mm_compaction_begin { time(); }'
    ```
    
    解释：记录内存压缩事件的时间戳。
    

---

### **工具对比**

- **BCC**
    - 提供封装好的工具，使用门槛较低。
    - 注重性能，适合快速实现特定功能。
    - 适用于对某些特定任务的深入分析。
- **bpftrace**
    - 语法灵活，能够快速开发脚本。
    - 更适合动态、复杂的分析任务。
    - 比较适合有一定编程基础的用户。

## OPTIONAL EXERCISES

#### **1. 运行 `vmscan(8)`，并监测直接回收时间**

- **目标**：使用 `vmscan(8)` 查看内存回收情况。如果观察到直接回收 (`D-RECLAIMms`)，进一步运行 `drsnoop(8)`。
- **实现**：
    - 使用 `vmscan(8)` 运行 10 分钟：
        
        ```bash
        vmscan
        ```
        
    - 若存在直接回收，运行：
        
        ```bash
        drsnoop
        ```
        
- **关键点**：监控内存压力是否导致直接回收事件频繁发生，这通常表明系统内存不足或内存分配策略需要优化。

---

#### **2. 修改 `vmscan(8)` 每 20 行打印一次表头**

- **目标**：增强可读性，使表头在长时间运行中不会滚动消失。
- **实现**：
    - 修改 BCC 源码或脚本，添加计数器逻辑，检查每 20 行输出时重打印表头。
    - 示例伪代码：
        
        ```python
        line_count = 0
        for line in vmscan_output:
            if line_count % 20 == 0:
                print_header()
            print(line)
            line_count += 1
        ```
        

---

#### **3. 使用 `fault(8)` 在应用启动期间统计页面错误栈跟踪**

- **目标**：在应用启动阶段捕获页面错误的栈跟踪。
- **实现**：
    - 启动应用时运行 `fault(8)`：
        
        ```bash
        fault
        ```
        
    - 确保应用支持符号表解析（需要调试符号 `-g`）。
- **用途**：分析应用初始化过程中频繁发生页面错误的原因，优化内存预取或页面访问模式。

---

#### **4. 基于 (3) 的输出创建页面错误火焰图**

- **目标**：可视化页面错误栈，定位热点。
- **实现**：
    - 使用 `flamegraph.pl` 脚本生成火焰图：
        
        ```bash
        fault > pagefaults.txt
        ./stackcollapse-bpftrace.pl pagefaults.txt | ./flamegraph.pl > pagefaults.svg
        ```
        
    - 打开 `pagefaults.svg` 查看结果。
- **关键点**：火焰图帮助直观识别高频页面错误发生的位置。

---

#### **5. 开发工具跟踪通过 `brk(2)` 和 `mmap(2)` 的虚拟内存增长**

- **目标**：监测进程虚拟内存分配的两种主要方式。
- **实现**：
    - 使用 tracepoints 或 kprobes：
        
        ```bash
        bpftrace -e '
        tracepoint:syscalls:sys_enter_brk,
        tracepoint:syscalls:sys_enter_mmap { @[comm] = count(); }'
        ```
        
    - 打印每次调用的内存大小（通过参数解析）。
- **用途**：了解程序内存分配模式，评估分配策略。

---

#### **6. 开发工具打印 `brk(2)` 扩展大小**

- **目标**：捕获每次 `brk(2)` 系统调用扩展的具体大小。
- **实现**：
    - 使用 `bpftrace`：
        
        ```bash
        bpftrace -e '
        tracepoint:syscalls:sys_enter_brk { printf("PID %d expanded by %d bytes\n", pid, args->brk_size); }'
        ```
        
- **用途**：识别是否存在不合理的小块内存分配，优化分配行为。

---

#### **7. 开发工具显示页面压缩所花时间**

- **目标**：测量页面压缩事件的时间，并以直方图呈现。
- **实现**：
    - 捕获开始和结束事件：
        
        ```bash
        bpftrace -e '
        tracepoint:compaction:mm_compaction_begin { @start[pid] = nsecs; }
        tracepoint:compaction:mm_compaction_end { 
          @time = hist(nsecs - @start[pid]); delete(@start[pid]); }'
        ```
        
    - 输出时间直方图。
- **用途**：分析页面压缩是否导致性能瓶颈。

---

#### **8. 显示 slab 回收时间，按 slab 名称分类**

- **目标**：跟踪 `shrink_slab` 执行时间并按 slab 名称分类。
- **实现**：
    - 使用 tracepoints 或 kprobes：
        
        ```bash
        bpftrace -e '
        tracepoint:mm:shrink_slab_start, tracepoint:mm:shrink_slab_end {
          @slab = hist(end_time - start_time); }'
        ```
        
- **用途**：分析特定 slab 是否回收耗时过长，优化内存管理。

---

#### **9. 使用 `memleak(8)` 查找内存泄漏**

- **目标**：发现长期存在的内存泄漏对象并评估工具性能开销。
- **实现**：
    - 运行测试环境程序，同时使用：
        
        ```bash
        memleak
        ```
        
    - 比较运行性能有无显著下降。
- **用途**：帮助定位泄漏问题的具体位置和严重程度。

---

#### **10. （高级挑战）开发工具分析交换抖动时间**

- **目标**：测量页面从换出到换入的时间分布。
- **实现**：
    - 捕获 swap out/in 事件的时间戳并计算差值。
    - 示例：
        
        ```bash
        bpftrace -e '
        tracepoint:mm:swap_out, tracepoint:mm:swap_in { 
          @swap_time = hist(nsecs_out - nsecs_in); }'
        ```
        
- **用途**：识别导致交换抖动的根本原因，优化交换机制。