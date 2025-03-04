  

# Appendix B

sar Summary

This is a summary of options and metrics from the system activity reporter, sar(1). You can use this to jog your memory about which metrics are under which options. See the man page for the full list.

sar(1) is introduced in [[ch04.xhtml#ch04|Chapter 4]], [[ch04.xhtml#ch04|Observability Tools]], [[ch04.xhtml#ch04lev4|Section 4.4]], and selected options are summarized in later chapters ([^6], [^7], [^8], [^9], [^10]).

**Option**

**Metrics**

**Description**

`-u-P ALL`

`**%user** %nice **%system %iowait %steal %idle**`

Per-CPU utilization (`-u` optional)

`-u`

`**%user** %nice **%system %iowait %steal %idle**`

CPU utilization

`-u ALL`

`**... %irq %soft** %guest %gnice`

CPU utilization extended

`-m CPU-P ALL`

`**MHz**`

Per-CPU frequency

`-q`

`**runq-sz** plist-sz ldavg-1 ldavg-5 ldavg-15 **blocked**`

CPU run-queue size

`-w`

`**proc/s cswch/s**`

CPU scheduler events

`-B`

`pgpgin/s pgpgout/s fault/s majflt/s pgfree/s **pgscank/s pgscand/s** pgsteal/s %vmeff`

Paging statistics

`-H`

`kbhugfree kbhugused %hugused`

Huge pages

`-r`

`kbmemfree **kbavail** kbmemused %memused kbbuffers kbcached kbcommit %commit kbactive kbinact kbdirty`

Memory utilization

`-S`

`kbswpfree kbswpused **%swpused** kbswpcad %swpcad`

Swap utilization

`-W`

`**pswpin/s pswpout/s**`

Swapping statistics

`-v`

`dentunusd file-nr inode-nr pty-nr`

Kernel tables

`-d`

`**tps rkB/s wkB/s** areq-sz aqu-sz **await** svctm **%util**`

Disk statistics

`-n DEV`

`**rxpck/s txpck/s rxkB/s txkB/s** rxcmp/s txcmp/s rxmcst/s **%ifutil**`

Network interface statistics

`-n EDEV`

`**rxerr/s txerr/s coll/s rxdrop/s txdrop/s** txcarr/s rxfram/s rxfifo/s txfifo/s`

Network interface errors

`-n IP`

`irec/s fwddgm/s idel/s orq/s asmrq/s asmok/s fragok/s fragcrt/s`

IP statistics

`-n EIP`

`ihdrerr/s iadrerr/s iukwnpr/s idisc/s odisc/s onort/s asmf/s fragf/s`

IP errors

`-n TCP`

`**active/s passive/s iseg/s oseg/s**`

TCP statistics

`-n ETCP`

`atmptf/s estres/s **retrans/s** isegerr/s orsts/s`

TCP errors

`-n SOCK`

`totsck tcpsck udpsck rawsck ip-frag tcp-tw`

Socket statistics

I have highlighted in bold the key metrics that I look for.

Some sar(1) options may require kernel features enabled (e.g., huge pages), and some metrics were added in later versions of sar(1) (version 12.0.6 is shown here).