# 68 - JVM Thread Management

Tags: #jvm-internals #section12
Related: [[69 - Thread Dump Analysis]] | [[70 - Debugging Multithreading]] | [[78 - Virtual Threads (Project Loom)]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand how the JVM creates, manages, and monitors threads internally — platform threads, virtual threads, and JVM-owned background threads.

---

## Beginner Explanation
When you call `thread.start()`, the JVM creates a real OS thread. The JVM also runs several of its own background threads (GC, JIT, etc.) that you never create. Understanding this helps you interpret thread dumps and tune your application.

---

## Technical Deep Dive

### Platform threads (Java ≤ 20 and Java 21+)
- 1 Java thread = 1 OS thread
- Created via `Thread.start()` → JVM calls `pthread_create` (Linux) or `CreateThread` (Windows)
- OS manages scheduling
- Stack size: typically 512KB–2MB (configurable with `-Xss`)
- Limit: typically ~thousands per JVM process before memory/OS limits hit

### Virtual threads (Java 21+) → [[78 - Virtual Threads (Project Loom)]]
- Millions of virtual threads mapped to a small pool of carrier (OS) threads
- Managed by the JVM scheduler, not the OS
- Tiny stack (grows on demand, starts at ~1KB)

### JVM system threads
When you start any Java program, the JVM creates these background threads:

| Thread name | Purpose |
|---|---|
| `main` | Executes `main()` |
| `GC Thread#N` | Parallel GC worker threads |
| `G1 Main Marker` | G1 concurrent marking |
| `Finalizer` | Runs `finalize()` on GC'd objects |
| `Reference Handler` | Processes weak/soft/phantom references |
| `Signal Dispatcher` | Handles OS signals (SIGTERM, etc.) |
| `Attach Listener` | Allows tools like `jstack` to attach |
| `CompilerThread#N` | JIT compiler threads (C1/C2) |
| `VM Periodic Task Thread` | JVM housekeeping |
| `Common-Cleaner` | Runs `Cleaner` actions (NIO, etc.) |

### Thread stack memory
Each platform thread needs a native stack. For 1000 threads with 1MB stack each: **1GB of stack memory** — not counted in `-Xmx`.

```
# Configure stack size
-Xss512k   # 512KB stack per thread (reduces memory per thread)
-Xss2m     # 2MB (allows deeper recursion)
```

### Monitoring JVM threads programmatically

```java
import java.lang.management.*;

ThreadMXBean tmx = ManagementFactory.getThreadMXBean();
System.out.println("Live threads: " + tmx.getThreadCount());
System.out.println("Peak threads: " + tmx.getPeakThreadCount());
System.out.println("Daemon threads: " + tmx.getDaemonThreadCount());
System.out.println("Total started: " + tmx.getTotalStartedThreadCount());

// Deadlock detection
long[] deadlocked = tmx.findDeadlockedThreads();
if (deadlocked != null) {
    ThreadInfo[] infos = tmx.getThreadInfo(deadlocked, true, true);
    // print stack traces
}
```

---

## Code

```java
import java.lang.management.*;
import java.util.*;

public class JVMThreadMgmtDemo {
    public static void main(String[] args) throws InterruptedException {
        ThreadMXBean tmx = ManagementFactory.getThreadMXBean();

        // Enable CPU time measurement
        if (tmx.isThreadCpuTimeSupported()) {
            tmx.setThreadCpuTimeEnabled(true);
        }

        // Print all JVM threads
        System.out.println("=== JVM Threads ===");
        Map<Thread, StackTraceElement[]> all = Thread.getAllStackTraces();
        all.entrySet().stream()
            .sorted(Comparator.comparing(e -> e.getKey().getName()))
            .forEach(e -> System.out.printf("%-40s daemon=%-5b state=%s%n",
                e.getKey().getName(), e.getKey().isDaemon(), e.getKey().getState()));

        System.out.println("\nTotal thread count: " + tmx.getThreadCount());
        System.out.println("Peak:   " + tmx.getPeakThreadCount());
        System.out.println("Daemon: " + tmx.getDaemonThreadCount());

        // CPU time per thread
        System.out.println("\n=== CPU Time ===");
        for (long id : tmx.getAllThreadIds()) {
            ThreadInfo ti = tmx.getThreadInfo(id);
            if (ti != null) {
                long cpu = tmx.getThreadCpuTime(id); // nanoseconds
                System.out.printf("%-40s cpu=%d ms%n", ti.getThreadName(), cpu / 1_000_000);
            }
        }

        // Check for deadlocks
        long[] deadlocked = tmx.findDeadlockedThreads();
        System.out.println("\nDeadlocked threads: " + (deadlocked == null ? "none" : Arrays.toString(deadlocked)));
    }
}
```

---

## Interview Questions

**Q: How does `Thread.start()` work at the JVM level?**
> `Thread.start()` calls the native method `Thread.start0()`, which invokes `pthread_create` (Linux) or `CreateThread` (Windows) to create a real OS thread. The OS thread runs the JVM's entry point, which calls `Thread.run()`.

**Q: What are JVM system threads and can you name some?**
> JVM creates background threads for: GC (`GC Thread#N`), JIT compilation (`CompilerThread#N`), finalization (`Finalizer`), signal handling (`Signal Dispatcher`), reference processing (`Reference Handler`), and tool attachment (`Attach Listener`).

**Q: How do you find deadlocked threads programmatically?**
> Use `ManagementFactory.getThreadMXBean().findDeadlockedThreads()` — returns thread IDs of threads in a deadlock cycle.

---

## Cheat Sheet
| API | Description |
|---|---|
| `tmx.getThreadCount()` | Current live thread count |
| `tmx.getPeakThreadCount()` | Highest ever reached |
| `tmx.findDeadlockedThreads()` | Returns deadlocked thread IDs |
| `Thread.getAllStackTraces()` | Map of all threads to stack traces |
| `-Xss512k` | Per-thread stack size |
| Virtual threads | Java 21+ — millions, ~1KB each |
