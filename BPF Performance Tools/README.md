# BPF Performance Tools
https://github.com/brendangregg/bpf-perf-tools-book?tab=readme-ov-file

正在学习BPF Performance Tools 这本书，我发给你具体的段落，请你用使用中文为我解释和分析，帮助我更好地理解书中的内容并做笔记。

> 我正在学习《BPF Performance Tools 》，这是书中的一段内容：  
> “How uprobes Work  
The approach is similar to kprobes: a fast breakpoint is  
inserted at the target instruction, which passes execution to a  
uprobe handler. When the uprobe is no longer needed, the  
target instructions are returned to their original state. With  
uretprobes, the function entry is instrumented with a uprobe,  
and the return address hijacked with a trampoline function, as  
with kprobes.  
We can see this in action using a debugger. Disassembling the  
readline() function from the bash(1) shell:  
At the time of writing, I still tend to use Ftrace for this particular task, since it  
is quicker to initialize and tear down instrumentation. See my funccount(8) tool  
from my Ftrace perf-tools repository. As of this writing, there is work under way  
to improve the speed of BPF kprobe initialization and tear down by batching  
operations. I hope it will be available by the time you are reading this.  
> 请帮我用中文解答这段内容，解释它的核心概念，并补充相关知识点，方便我理解和做笔记。