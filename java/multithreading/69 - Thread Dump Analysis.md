# 69 - Thread Dump Analysis

Tags: #jvm-internals #section12 #debugging
Related: [[68 - JVM Thread Management]] | [[70 - Debugging Multithreading]] | [[36 - Deadlock]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Learn how to capture, read, and interpret Java thread dumps — the single most powerful tool for debugging concurrency problems in production.

---

## Beginner Explanation
A thread dump is a snapshot of every thread's current state and stack trace at a moment in time. It tells you exactly what every thread was doing when you captured it — invaluable for diagnosing deadlocks, slow responses, and thread starvation.

---

## Analogy
A thread dump is like a freeze-frame photo of your entire team in the middle of their workday. You can see exactly who is working, who is waiting, who is blocked, and who is stuck in a loop.

---

## Technical Deep Dive

### How to capture a thread dump

| Method | Command / Action |
|---|---|
| `jstack` (CLI) | `jstack <pid>` |
| `jcmd` (preferred CLI) | `jcmd <pid> Thread.print` |
| Kill signal (Linux) | `kill -3 <pid>` → output to stdout |
| JVisualVM (GUI) | Thread → Thread Dump button |
| Java Flight Recorder | Continuous recording |
| Programmatic | `ManagementFactory.getThreadMXBean().dumpAllThreads(true, true)` |

### Anatomy of a thread dump entry

```
"http-nio-8080-exec-1" #34 prio=5 os_prio=0 cpu=1234ms elapsed=45s tid=0x00007f... nid=0x2a3b
   java.lang.Thread.State: BLOCKED (on object monitor)
   
   at com.example.OrderService.processOrder(OrderService.java:87)
   - waiting to lock <0x00000007c12345ab> (a java.lang.Object)
   at com.example.RequestHandler.handle(RequestHandler.java:42)
   at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.run(...)
   
   Locked ownable synchronizers:
   - <0x00000007c12349cd> (a java.util.concurrent.locks.ReentrantLock)
```

| Field | Meaning |
|---|---|
| Thread name | `"http-nio-8080-exec-1"` |
| Thread ID | `#34` — JVM ID |
| Native ID | `nid=0x2a3b` — OS thread ID (hex) |
| CPU time | `cpu=1234ms` |
| State | `BLOCKED`, `WAITING`, `RUNNABLE`, etc. |
| `waiting to lock` | Trying to enter `synchronized` block holding that object |
| `locked` | Locks this thread currently holds |

### Diagnosing common problems

**Deadlock**: look for "Found one Java-level deadlock" in the dump
```
Found one Java-level deadlock:
"Thread-1": waiting to lock <0xabc> (held by Thread-2)
"Thread-2": waiting to lock <0xdef> (held by Thread-1)
```

**Thread pool exhaustion**: all pool threads in BLOCKED/WAITING state
```
"pool-1-thread-1" ... WAITING
"pool-1-thread-2" ... WAITING
"pool-1-thread-3" ... WAITING
... all 200 threads waiting on same DB connection pool
```

**Hot lock (contention)**: many threads BLOCKED on the same monitor address `<0x...>`

**Infinite loop**: thread in RUNNABLE state, same stack frame across multiple dumps taken 10s apart

**Leaked thread**: thread in WAITING state with no apparent purpose — likely a thread pool not shut down

### Taking multiple dumps
Take 3 dumps, 10 seconds apart. Compare:
- **Same RUNNABLE frame** across all 3 → likely a CPU-spinning bug
- **Progressing stack** → healthy
- **BLOCKED same lock** across all 3 → lock contention or deadlock

---

## Code

```java
import java.lang.management.*;

public class ThreadDumpProgrammatic {
    public static void main(String[] args) {
        ThreadMXBean tmx = ManagementFactory.getThreadMXBean();
        ThreadInfo[] threads = tmx.dumpAllThreads(true, true); // with locks

        for (ThreadInfo ti : threads) {
            System.out.printf("%n=== %s [%s] ===%n", ti.getThreadName(), ti.getThreadState());

            // Show what lock it's waiting for
            if (ti.getLockName() != null) {
                System.out.println("  Waiting for: " + ti.getLockName()
                    + " held by: " + ti.getLockOwnerName());
            }

            // Show held locks
            for (LockInfo li : ti.getLockedSynchronizers()) {
                System.out.println("  Holds: " + li.getClassName());
            }

            // Stack trace (top 5 frames)
            StackTraceElement[] stack = ti.getStackTrace();
            for (int i = 0; i < Math.min(5, stack.length); i++) {
                System.out.println("    at " + stack[i]);
            }
        }

        // Deadlock detection
        long[] deadlocked = tmx.findDeadlockedThreads();
        if (deadlocked != null) {
            System.out.println("\n!!! DEADLOCK DETECTED in threads: ");
            for (ThreadInfo ti : tmx.getThreadInfo(deadlocked)) {
                System.out.println("  " + ti.getThreadName());
            }
        }
    }
}
```

---

## Interview Questions

**Q: How do you capture a thread dump in production without restarting the JVM?**
> `jstack <pid>`, `jcmd <pid> Thread.print`, or `kill -3 <pid>`. None of these require a restart.

**Q: What does BLOCKED vs WAITING look like in a thread dump?**
> BLOCKED: `waiting to lock <0x...> (on object monitor)`. WAITING: `in Object.wait()` or `parking to wait for...`. BLOCKED = fighting for a `synchronized` lock. WAITING = voluntarily waiting for a signal.

**Q: How do you identify a deadlock in a thread dump?**
> Look for the "Found one Java-level deadlock" section. Or manually find threads where Thread A is waiting for Lock X held by Thread B, which is waiting for Lock Y held by Thread A.

**Q: A thread dump shows all 200 Tomcat threads in WAITING state. What's wrong?**
> Likely thread pool exhaustion — all threads are blocked waiting on a downstream resource (DB connection, external API). The thread pool is empty so new requests queue up indefinitely.

---

## Cheat Sheet
| Tool | Command |
|---|---|
| jstack | `jstack <pid>` |
| jcmd | `jcmd <pid> Thread.print` |
| kill | `kill -3 <pid>` (Linux) |
| Deadlock marker | `Found one Java-level deadlock` |
| BLOCKED | `waiting to lock <0x...>` |
| WAITING | `in Object.wait()` or `parking` |
| Diagnosis tip | 3 dumps × 10s apart — compare states |
