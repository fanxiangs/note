## **练习题（Exercises）**

如果你想更深入地探索 eBPF 程序，可以尝试以下操作：

### **1. 使用 `ip link` 命令附加和分离 XDP 程序**
你可以使用 `ip link` 命令将 XDP（eXpress Data Path）程序附加到网络接口 `eth0`，然后再将其移除：

```sh
# 将 hello.bpf.o XDP 程序附加到 eth0
$ ip link set dev eth0 xdp obj hello.bpf.o sec xdp

# 关闭 XDP 程序
$ ip link set dev eth0 xdp off
```

---

### **2. 运行 Chapter 2 中的任意 BCC 示例**
运行 **BCC（BPF Compiler Collection）** 示例程序，并在 **另一个终端窗口** 使用 `bpftool` 检查已加载的 eBPF 程序。例如，运行 `hello-map.py` 并查看其状态：

```sh
$ bpftool prog show name hello
```

示例输出：
```
197: kprobe  name hello  tag ba73a317e9480a37  gpl
        loaded_at 2022-08-22T08:46:22+0000  uid 0
        xlated 296B  jited 328B  memlock 4096B  map_ids 65
        btf_id 179
        pids hello-map.py(2785)
```

你还可以使用 `bpftool prog dump` 命令查看字节码（bytecode）和 JIT 机器码（machine code）版本：

```sh
$ bpftool prog dump xlated name hello
```

---

### **3. 运行 `hello-tail.py` 并查看 tail call 相关的 eBPF 程序**
运行 `chapter2` 目录中的 `hello-tail.py`，然后在另一个终端查看它加载的所有 tail call 程序：

```sh
$ bpftool prog list
```

示例输出：
```
120: raw_tracepoint  name hello  tag b6bfd0e76e7f9aac  gpl
        loaded_at 2023-01-05T14:35:32+0000  uid 0
        xlated 160B  jited 272B  memlock 4096B  map_ids 29
        btf_id 124
        pids hello-tail.py(3590)

121: raw_tracepoint  name ignore_opcode  tag a04f5eef06a7f555  gpl
        loaded_at 2023-01-05T14:35:32+0000  uid 0
        xlated 16B  jited 72B  memlock 4096B
        btf_id 124
        pids hello-tail.py(3590)

122: raw_tracepoint  name hello_exec  tag 931f578bd09da154  gpl
        loaded_at 2023-01-05T14:35:32+0000  uid 0
        xlated 112B  jited 168B  memlock 4096B
        btf_id 124
        pids hello-tail.py(3590)

123: raw_tracepoint  name hello_timer  tag 6c3378ebb7d3a617  gpl
        loaded_at 2023-01-05T14:35:32+0000  uid 0
        xlated 336B  jited 356B  memlock 4096B
        btf_id 124
        pids hello-tail.py(3590)
```

你还可以使用 `bpftool prog dump xlated` 命令查看字节码，并将其与 **“BPF to BPF Calls”** 中的内容进行对比：

```sh
$ bpftool prog dump xlated name hello_timer
```

---

### **4. **实验 `XDP_ABORTED` 的影响（小心！）**
**请谨慎操作！** **最好只是思考这个问题，而不要直接尝试！** 

在 XDP 程序中，如果返回值为 `0`，则表示 **`XDP_ABORTED`**，内核会立即丢弃该数据包，停止进一步处理。这一点与 C 语言中的 `0` 通常表示成功相反。

如果你修改 XDP 程序的返回值为 `0`，并将其附加到 **虚拟机（VM）的 `eth0` 接口**，那么 **所有网络数据包都会被丢弃**，包括 SSH 连接。这可能会导致你无法访问该机器，需要手动重启才能恢复。

你可以考虑在 **Docker 容器** 中运行 XDP 程序，让它只影响 **容器内的虚拟网卡**，而不会影响整个虚拟机。可以参考这个示例：
👉 [GitHub: lb-from-scratch](https://github.com/lizrice/lb-from-scratch)

---

## **总结**
1. **附加和移除 XDP 程序**：使用 `ip link` 命令进行实验。
2. **运行 BCC 示例**：在运行的同时使用 `bpftool` 检查加载的程序和字节码。
3. **分析 tail call 机制**：运行 `hello-tail.py` 并查看它加载的所有程序。
4. **谨慎实验 XDP_ABORTED 行为**：如果返回 `0`，将丢弃所有数据包，小心导致 SSH 断连。

这些练习可以帮助你更深入地理解 eBPF 及其调试工具。 🚀