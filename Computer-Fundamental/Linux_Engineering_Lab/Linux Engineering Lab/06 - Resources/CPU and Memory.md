---
type: concept
status: new
level: 3
tags: [linux, cpu, memory, jvm]
---

# CPU and Memory

## Mental model
- **CPU**: time slices on cores. A runnable queue longer than the core count means waiting.
- **Load average**: runnable + uninterruptible-wait tasks (1, 5, 15 min). Compare with core count (`nproc`). Not CPU %.
- **Memory**: RAM is used for process memory *and* page cache. Linux deliberately fills spare RAM with cache. Look at **available**, not **free**.

## Inspect
```bash
uptime
nproc
free -h
vmstat 1 5
top          # press 1 for per-core, M sort by memory, P by CPU
ps -eo pid,user,%cpu,%mem,rss,etime,cmd --sort=-rss | head
cat /proc/meminfo | head
```

## Reading `free -h`
```
              total   used   free  shared  buff/cache  available
Mem:           7.7Gi  2.1Gi  300Mi   50Mi       5.3Gi      5.2Gi
```
Only 300 MiB "free" is fine here: **available 5.2 GiB** is what matters.

## vmstat clues
| Column | Meaning | Concern |
|---|---|---|
| `r` | runnable tasks | consistently > cores |
| `b` | blocked on I/O | sustained > 0 |
| `si/so` | swap in/out | sustained activity |
| `wa` | CPU waiting for I/O | high = storage problem, not CPU |

## Out of memory
The kernel's OOM killer picks a victim when memory is exhausted. Evidence:
```bash
journalctl -k | grep -i -E 'out of memory|killed process'
systemctl show UNIT -p ExecMainStatus     # often 137
```
In containers, cgroup limits can trigger OOM **below** host capacity.

## JVM caveat
Heap (`-Xmx`) is only part of a Java process's memory. Add metaspace, thread stacks, direct buffers, code cache, GC structures. A 2 GB heap can mean a 3 GB+ resident process. Size container/host limits accordingly.

## Common mistakes
- Panicking over low "free".
- Equating load with CPU.
- Setting `-Xmx` equal to the container memory limit.
- Ignoring `wa` and chasing CPU.

## Active recall
Which `free` column matters most?:: `available`.
High load but low CPU use suggests?:: Tasks blocked on I/O (D-state), check storage.
Why can a Java process exceed `-Xmx` in RSS?:: Non-heap memory: metaspace, stacks, direct buffers, native.
What does exit 137 often mean?:: SIGKILL, frequently the OOM killer.

## My explanation
> Explain why "free memory is low" is not by itself a problem on Linux:

## Next
→ [[Storage and Filesystems]]
