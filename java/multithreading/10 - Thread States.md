# Thread States

## Objective

By the end of this note, you will understand:

- How Java's `Thread.State` values map to real **OS-level** thread states
- How to recognize each state in a real **thread dump** (`jstack` output)
- How monitoring tools (JConsole, VisualVM, Java Flight Recorder) visualize thread states
- How to use state information to diagnose real production issues

This note assumes you've read [[09 - Thread Lifecycle]], which covers the full state machine and transition triggers. This note goes one level deeper: **how these states actually show up in real tools**, which is the skill you'll use during an actual production incident.

---

## Why should I learn this?

Knowing the theoretical state machine (from [[09 - Thread Lifecycle]]) is necessary but not sufficient. In a real incident, you won't be calling `.getState()` in code — you'll be staring at a `jstack` dump with hundreds of threads, or a monitoring dashboard, trying to figure out **what's actually wrong**. This note bridges the gap between "I know the six states" and "I can read a production thread dump and diagnose the problem in under 5 minutes."

---

## Beginner Explanation

Imagine a security camera system watching every employee (thread) in a large office. At any given moment, you can pull up a snapshot showing exactly what every employee is doing: typing at their desk (running), standing in line for the printer (blocked), on a scheduled coffee break (timed waiting), or waiting for a colleague to reply before continuing (waiting).

A **thread dump** is exactly this: a snapshot, frozen in time, showing what every single thread in your JVM is doing **right now**, including the exact line of code (stack trace) where it currently sits.

---

## Real World Analogy

**Warehouse Analogy**

Picture a warehouse manager who wants to understand why order fulfillment has slowed down. They don't just want a summary — they want to see, for **every single worker**, exactly what they're doing at this exact moment:

- Actively packing a box (`RUNNABLE`)
- Standing at a shared workbench waiting for their turn (`BLOCKED`)
- Waiting for a supervisor's signal to start the next task (`WAITING`)
- On a 15-minute scheduled break (`TIMED_WAITING`)

By walking the floor and noting every worker's exact status, the manager can immediately spot the bottleneck — say, 20 workers all waiting at the same workbench. This "walk the floor and check everyone's status" action is exactly what a thread dump does for a running JVM.

---

## Technical Explanation

### Java `Thread.State` vs. OS Thread States

Java's `Thread.State` enum is a **JVM-level abstraction** — it does not map 1:1 to the underlying OS thread states, which vary by operating system (Linux, Windows, macOS each have their own scheduler state names). Here's a rough mapping for a Linux-based JVM (the most common production environment):

|Java `Thread.State`|Typical Underlying OS State (Linux)|
|---|---|
|NEW|No OS thread exists yet|
|RUNNABLE|`R` (Running) or ready-to-run in scheduler queue|
|BLOCKED|`D` (Uninterruptible sleep) or futex-wait, specific to lock acquisition|
|WAITING|`S` (Interruptible sleep), parked via futex|
|TIMED_WAITING|`S` (Interruptible sleep), with a timer set|
|TERMINATED|OS thread has exited; may briefly remain as a zombie until cleaned up|

The key takeaway: **Java's abstraction hides OS-level complexity**, giving you a simplified, portable view — but tools that go deeper (like `perf` on Linux) can show you the raw OS thread state if needed.

### Reading a Thread Dump (`jstack` output)

A `jstack <pid>` command produces output like this for each thread:

```
"worker-thread-1" #14 prio=5 os_prio=0 tid=0x00007f... nid=0x4a07 waiting on condition [0x00007f...]
   java.lang.Thread.State: TIMED_WAITING (sleeping)
        at java.lang.Thread.sleep(Native Method)
        at com.example.OrderProcessor.run(OrderProcessor.java:42)
        at java.lang.Thread.run(Thread.java:834)
```

Key parts to read:

- **Thread name** (`"worker-thread-1"`) — this is why [[16 - Thread Naming]] matters so much; an unnamed thread shows as `Thread-14`, giving you zero context during an incident.
- **`java.lang.Thread.State`** — the exact state, straight from the enum covered in [[09 - Thread Lifecycle]].
- **Stack trace** — the exact method and line number where the thread currently sits, which tells you **why** it's in that state.

### Recognizing a Deadlock in a Thread Dump

`jstack` output often includes an explicit deadlock detection section at the bottom:

```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f... (object 0x000000076... , a java.lang.Object),
  which is held by "Thread-2"
"Thread-2":
  waiting to lock monitor 0x00007f... (object 0x000000076... , a java.lang.Object),
  which is held by "Thread-1"
```

This is `jstack` doing the cycle-detection work for you — recognizing that `Thread-1` is `BLOCKED` waiting on a lock held by `Thread-2`, which is itself `BLOCKED` waiting on a lock held by `Thread-1`. Full detail on deadlocks in [[36 - Deadlock]].

### Monitoring Tool Visualizations

- **JConsole / VisualVM**: Show a live-updating bar or timeline per thread, color-coded by state (commonly green = running, yellow = waiting/blocked, etc.), letting you visually spot when many threads suddenly turn a "blocked" color simultaneously.
- **Java Flight Recorder (JFR)**: Records detailed thread state transitions over time, letting you replay exactly when and why a thread entered a given state, which is invaluable for diagnosing intermittent production issues that are hard to catch live.

---

## Internal Working

How a thread dump is actually captured, step by step:

1. You run `jstack <pid>` (or trigger a dump via `kill -3 <pid>` on Linux/macOS, or Ctrl+Break on Windows).
2. The JVM receives this signal/request and performs a **safepoint** — briefly pausing all application threads at a safe point in their execution (this is why taking a thread dump can cause a very brief pause in a running application).
3. For each live thread, the JVM records:
    - Its current `Thread.State`
    - Its full call stack (method names, class names, line numbers)
    - Any locks it currently holds or is waiting to acquire
4. The JVM analyzes lock-wait relationships between threads to detect potential deadlock cycles.
5. All of this is written out as human-readable text output.

---

## Memory Diagram

```
                     JVM Process
      ---------------------------------------------
      |  jstack <pid> triggers a safepoint pause    |
      |                                              |
      |   Thread-1: RUNNABLE   [stack trace...]      |
      |   Thread-2: BLOCKED    [stack trace...]      |
      |              (waiting on lock held by T-3)   |
      |   Thread-3: TIMED_WAITING [stack trace...]   |
      |   Thread-4: WAITING    [stack trace...]      |
      ---------------------------------------------
                        |
                        v
              Full snapshot printed to
              console / file for analysis
```

---

## Execution Flow

```
Production incident: response times spiking
        |
        v
Engineer runs: jstack <pid> > dump.txt
        |
        v
JVM pauses briefly, captures all thread states + stacks
        |
        v
Engineer searches dump.txt for:
  - Large clusters of BLOCKED threads (lock contention)
  - Threads stuck in the SAME line of code across many samples
  - An explicit "Found one Java-level deadlock" section
        |
        v
Root cause identified (e.g., 40 threads BLOCKED
waiting on a single database connection pool lock)
```

---

## Code Example

While thread dumps are typically taken externally via `jstack`, you can also programmatically inspect all live threads from within your own Java code — useful for building custom health checks or diagnostics:

```java
import java.lang.management.ManagementFactory;
import java.lang.management.ThreadInfo;
import java.lang.management.ThreadMXBean;

public class ThreadStateInspector {

    public static void main(String[] args) throws InterruptedException {

        // Start a thread that will end up BLOCKED
        Object lock = new Object();

        Thread holder = new Thread(() -> {
            synchronized (lock) {
                try {
                    Thread.sleep(3000); // hold the lock for 3 seconds
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }, "Lock-Holder");

        Thread waiter = new Thread(() -> {
            synchronized (lock) {
                System.out.println("Waiter finally got the lock");
            }
        }, "Lock-Waiter");

        holder.start();
        Thread.sleep(500); // ensure holder grabs the lock first
        waiter.start();

        Thread.sleep(500); // give the waiter time to become BLOCKED

        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        ThreadInfo[] allThreads = threadMXBean.dumpAllThreads(false, false);

        for (ThreadInfo info : allThreads) {
            if (info.getThreadName().equals("Lock-Waiter") || info.getThreadName().equals("Lock-Holder")) {
                System.out.println(info.getThreadName() + " -> " + info.getThreadState());
            }
        }

        holder.join();
        waiter.join();
    }
}
```

### Line-by-line explanation

1. `ThreadMXBean` — a JMX (Java Management Extensions) interface that gives programmatic access to live thread information, exactly the kind of data `jstack` extracts, but accessible from within your own running application.
2. `threadMXBean.dumpAllThreads(false, false)` — captures a snapshot of every live thread's info (the `false, false` parameters control whether to also capture lock ownership and synchronizer details).
3. The loop prints `Lock-Holder`'s and `Lock-Waiter`'s states — you should see `Lock-Holder` as `TIMED_WAITING` (sleeping while holding the lock) and `Lock-Waiter` as `BLOCKED` (waiting to acquire that same lock).

---

## Dry Run

|Time|Lock-Holder State|Lock-Waiter State|
|---|---|---|
|0ms|Starts, acquires lock, sleeps|Not yet started|
|500ms|`TIMED_WAITING` (sleeping, holding lock)|Starts, tries to acquire lock|
|1000ms|`TIMED_WAITING` (still sleeping)|`BLOCKED` (waiting for lock)|
|3000ms|Releases lock, finishes|Acquires lock, runs, finishes|

---

## Common Mistakes

1. **Taking a single thread dump and assuming it tells the whole story.**
    
    - A single snapshot can be misleading — a thread might just happen to be `BLOCKED` for a normal, brief moment. Production diagnosis should take **multiple dumps a few seconds apart** and look for threads stuck in the **same** state/stack trace across all of them — that's the real signal of a genuine problem.
2. **Ignoring thread names in dumps.**
    
    - Generic names like `Thread-14` or `pool-2-thread-7` make it much harder to identify which part of your application a stuck thread belongs to. Always name threads meaningfully in production code (see [[16 - Thread Naming]]).
3. **Confusing high CPU usage with lock contention.**
    
    - Many `RUNNABLE` threads with high CPU usage indicates a **computation-bound** bottleneck; many `BLOCKED` threads with low CPU usage indicates a **lock contention** bottleneck. These require very different fixes, and thread dumps let you tell them apart clearly.
4. **Not realizing that taking a thread dump has a (usually brief) performance cost.**
    
    - Because it requires a JVM safepoint pause, taking thread dumps very frequently (e.g., every second) in a high-throughput production system can itself add measurable overhead.

---

## Production Usage

- **Incident response**: The very first action many senior Java engineers take during a "the service is slow/hanging" incident is capturing 3-5 thread dumps, a few seconds apart, via `jstack`.
- **APM tools** (Application Performance Monitoring — e.g., Datadog, New Relic, Dynatrace) automatically sample thread states continuously and surface "hot" blocked threads or long-running stack traces in their dashboards.
- **Automated deadlock detection**: Some production systems run periodic `ThreadMXBean.findDeadlockedThreads()` checks as part of their health-check endpoints, proactively alerting before a full deadlock brings the system down.
- **Java Flight Recorder (JFR)**: Increasingly used in production (very low overhead) to continuously record thread state transitions, enabling after-the-fact analysis of intermittent issues that are hard to catch with manual `jstack` snapshots.

---

## Interview Questions

**Beginner:**

1. What is a thread dump, and what does it show you?
2. What command is commonly used to take a thread dump for a running Java process?
3. What does the `Thread.State` field in a thread dump entry tell you?

**Intermediate:** 4. Why is it recommended to take multiple thread dumps a few seconds apart, rather than just one? 5. How can you tell the difference between a CPU-bound bottleneck and a lock-contention bottleneck from a thread dump? 6. What does `jstack`'s deadlock detection section actually show you?

**Advanced:** 7. What is a JVM "safepoint," and why does taking a thread dump require one? 8. How does `ThreadMXBean` let you programmatically access thread state information from within your own application?

**Production-Level:** 9. Walk through how you would diagnose a production incident where response times have spiked, using thread dumps as your primary tool. 10. Why might APM tools prefer continuous low-overhead sampling (like JFR) over periodically triggering full `jstack` dumps?

---

## Follow-up Questions

- Does taking a thread dump stop your application from serving requests? _(It causes a brief pause due to the JVM safepoint, typically milliseconds, not a full stop — but under heavy load or with very large heaps, this pause can be more noticeable.)_
- Can you take a thread dump of a remote production server without SSH access? _(Yes, via JMX-based tools if remote JMX is enabled, or via APM agents that continuously collect this data without requiring manual intervention.)_
- If a thread shows as `RUNNABLE` in every dump you take over several minutes, but CPU usage for that thread is low, what might be happening? _(It could be in a native/blocking I/O call, like a socket read, which Java sometimes reports as `RUNNABLE` even though the OS-level thread is actually blocked waiting on I/O — a known quirk of Java's `Thread.State` abstraction.)_

---

## Best Practices

- Always name your threads meaningfully in production code — this single habit makes thread dumps dramatically more useful.
- Take multiple thread dumps in quick succession during an incident, not just one, to distinguish transient states from genuine stuck threads.
- Set up automated deadlock detection as part of routine health checks for critical production services.
- Prefer low-overhead continuous monitoring (JFR, APM agents) over manual dumps for catching intermittent, hard-to-reproduce issues.

---

## Performance Notes

- Taking a thread dump requires a JVM-wide safepoint, causing a brief application pause — the pause duration generally scales with the number of live threads and heap size.
- Continuous thread-state monitoring via JFR is designed to have extremely low overhead (often well under 1%), making it safe to run in production continuously, unlike frequent manual `jstack` dumps.

---

## Comparison Table

|Tool|Overhead|Best Used For|
|---|---|---|
|`jstack` (manual dump)|Brief safepoint pause|On-demand incident investigation|
|JConsole / VisualVM|Low, but requires live connection|Interactive, real-time exploration|
|Java Flight Recorder (JFR)|Very low, continuous|Long-term/production-safe continuous monitoring|
|`ThreadMXBean` (programmatic)|Low|Custom health checks, automated deadlock detection|

---

## Summary

- Java's `Thread.State` enum is an abstraction over more complex, OS-specific thread states.
- Thread dumps (`jstack`) capture a full snapshot of every thread's state and stack trace at a given moment, and are the primary tool for diagnosing concurrency-related production incidents.
- Reading thread dumps well requires recognizing patterns: many `BLOCKED` threads on the same lock (contention), many `RUNNABLE` threads with high CPU (compute-bound), or explicit deadlock cycles.
- `ThreadMXBean` provides programmatic access to the same information, enabling custom health checks and automated deadlock detection.
- Multiple dumps, taken a few seconds apart, are far more diagnostic than a single snapshot.

---

## Cheat Sheet

|Symptom in Thread Dump|Likely Cause|
|---|---|
|Many threads `BLOCKED` on the same lock|Lock contention bottleneck|
|Many threads `RUNNABLE`, high CPU|Compute-bound bottleneck|
|Explicit "Found one Java-level deadlock"|Real deadlock — needs immediate fix|
|Threads stuck at the same stack trace across dumps|Genuinely stuck, not transient|
|Many threads `WAITING`/`TIMED_WAITING` on a pool|Resource pool likely too small for load|

---

## Keywords

Thread Dump, jstack, Thread.State, ThreadMXBean, Safepoint, JFR (Java Flight Recorder), Deadlock Detection, JConsole, VisualVM

---

## Related Notes

[[09 - Thread Lifecycle]] [[16 - Thread Naming]] [[36 - Deadlock]] [[69 - Thread Dump Analysis]] [[71 - Profiling Threads]]

---

## Practice Exercises

**Easy:**

1. Write a simple Java program with a few named threads, run it, and take a `jstack` dump while it's running. Identify each thread's state in the output.
2. Use `ThreadMXBean.dumpAllThreads()` programmatically to print all live thread states from within a running program.

**Medium:** 3. Reproduce the Lock-Holder/Lock-Waiter example from this note, and take a real `jstack` dump while it's running — locate both threads and confirm their states match what the code predicts. 4. Intentionally create 2 threads that deadlock each other, take a `jstack` dump, and find the "Found one Java-level deadlock" section in the output.

**Hard:** 5. Build a small health-check method using `ThreadMXBean.findDeadlockedThreads()` that periodically checks for deadlocks and logs a warning if any are found — a simplified version of what production monitoring systems do.

---

## Mini Project

**Project: Mini Thread Dump Analyzer**

Build a small tool that:

1. Uses `ThreadMXBean` to capture all live thread states every 2 seconds, for 5 samples.
2. Stores each sample's thread-name-to-state mapping.
3. After 5 samples, analyzes the data: flags any thread that was in the **same** state across all 5 samples (a likely "stuck" thread) and prints a summary report.

**Goal:** Build (in miniature) the exact kind of "diff across multiple dumps" analysis real engineers do manually during incidents — reinforcing why multiple samples matter more than one snapshot.

---

## Common Interview Trap

**Trap Question:** _"If a thread is doing a blocking network read (like reading from a socket), what state will it show in a thread dump — `WAITING`, `BLOCKED`, or `RUNNABLE`?"_

The trap is that most candidates assume it must be `WAITING` or `BLOCKED`, since intuitively "blocked on I/O" sounds like a wait state. In reality, native/blocking I/O operations are frequently reported as **`RUNNABLE`** in Java's `Thread.State`, because the JVM's state model doesn't have fine-grained visibility into OS-level I/O blocking the way it does for `synchronized` locks or `wait()`/`sleep()`. This is a well-known quirk that catches out engineers relying purely on `Thread.State` to diagnose I/O-related slowness — you often need additional native/OS-level tooling (like `perf` or `strace` on Linux) to see the true picture for I/O-bound threads.

---

## Frequently Asked Questions

**Q: Is `jstack` safe to run on a production system?** A: Generally yes — it causes a brief safepoint pause, but this is typically fast enough (milliseconds) to be safe even in production, and it's a standard, widely-used diagnostic practice.

**Q: Does a thread dump show which locks a thread currently holds, not just which it's waiting for?** A: Yes — with the right flags (or using `ThreadMXBean.dumpAllThreads(true, true)` to include lock/synchronizer info), thread dumps show both held locks and locks being waited on, which is exactly how deadlock cycles are detected.

**Q: Can thread dumps show what's happening inside third-party libraries or the JDK itself?** A: Yes — the stack trace in a thread dump includes every frame, including JDK internals and third-party library code, as long as it's genuine Java code (native code frames show up but with less detail).

**Q: How is Java Flight Recorder different from a manual thread dump?** A: JFR continuously records events (including thread state transitions) over time with very low overhead, letting you analyze historical data after an issue occurred — whereas a manual `jstack` dump only captures a single moment, and only if you happen to trigger it while the problem is actively occurring.

---

## Revision Checklist

- [ ] I understand that Java's `Thread.State` is an abstraction over OS-level thread states.
- [ ] I can read a basic `jstack` thread dump entry and identify the thread's name, state, and stack trace.
- [ ] I understand how to recognize lock contention (many `BLOCKED` threads) versus CPU-bound issues (many `RUNNABLE` threads) in a dump.
- [ ] I know why taking multiple dumps is more informative than a single snapshot.
- [ ] I understand the I/O-blocking-shows-as-RUNNABLE quirk and why it matters for diagnosis.

---

## Flashcards

**Q:** What command is commonly used to take a Java thread dump? **A:** `jstack <pid>`

**Q:** What does a large cluster of `BLOCKED` threads on the same lock indicate? **A:** Lock contention — many threads competing for one shared resource.

**Q:** Why should you take multiple thread dumps instead of just one during an incident? **A:** To distinguish transient/normal states from threads genuinely stuck in the same place over time.

**Q:** What JVM mechanism does taking a thread dump require, and what's its side effect? **A:** A safepoint — it causes a brief pause in all application threads.

**Q:** What Java API provides programmatic access to thread state information? **A:** `ThreadMXBean` (via `java.lang.management`).

---

## References

- Oracle Java Tutorials — Troubleshooting Guide (Thread Dumps): https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/
- `java.lang.management.ThreadMXBean` API Documentation
- `jstack` Tool Documentation (JDK Tools Reference)