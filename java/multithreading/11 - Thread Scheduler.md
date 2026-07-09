# Thread Scheduler

## Objective

By the end of this note, you will understand:

- What the thread scheduler is and who actually controls it (hint: not the JVM)
- How the OS decides which thread runs next
- What thread priority actually does (and its real, limited influence)
- Why you cannot rely on scheduling behavior for program correctness

This note builds on [[09 - Thread Lifecycle]] and [[10 - Thread States]], and leads into [[12 - Context Switching]].

---

## Why should I learn this?

A huge number of beginner concurrency bugs come from a single false assumption: **"the JVM controls exactly when my threads run, so I can predict the order."** This is wrong. The actual scheduling decisions are made by the **operating system**, using algorithms your Java code has almost no control over.

Understanding this is critical because:

- It explains why your multithreaded program behaves differently on different machines, operating systems, or even different runs on the same machine
- It explains why relying on `Thread.sleep()` or priorities to "control" execution order is fundamentally fragile
- It's a common interview topic, especially the distinction between what Java can influence (priority, hints) versus what it cannot control (actual OS scheduling decisions)

---

## Beginner Explanation

Imagine an airport control tower deciding which plane gets to take off next. Pilots (threads) don't decide this themselves — they can request priority, but the **air traffic controller** (the OS scheduler) makes the final call, based on factors the pilots don't fully see: runway availability, other traffic, safety rules, and fairness policies.

In Java:

- Your code creates threads and can set a "priority" (a hint, like a pilot saying "I have an urgent flight").
- But the **actual decision** of which thread runs at any given moment is made by the **operating system's scheduler**, not by your Java code and not even by the JVM itself.
- This means: **you cannot reliably predict or control the exact order in which threads execute**, no matter how carefully you set priorities.

---

## Real World Analogy

**Traffic Signal Analogy**

Think of a busy 4-way traffic intersection with no traffic lights, just a traffic officer directing cars (threads) through one at a time.

- Every car wants to go, but only one (or a few, on a multi-lane road / multi-core CPU) can actually move at once.
- The officer (the OS scheduler) decides who goes next, based on internal rules: how long you've been waiting, whether you signaled urgency, general fairness, etc.
- Cars cannot force their way through by simply "wanting to go first" — they can only request priority and hope the officer honors it.
- Crucially, **different officers (different operating systems, or even different JVM/OS versions) may make different decisions** given the exact same situation — which is exactly why you cannot rely on consistent thread ordering across environments.

---

## Technical Explanation

### The Scheduler Lives at the OS Level, Not the JVM

When you call `thread.start()` in Java, the JVM requests a **native OS thread**. From that point forward, it is the **operating system's thread scheduler** — not the JVM — that decides:

- Which `RUNNABLE` thread actually gets CPU time next
- How long each thread runs before being preempted (time-sliced) to let another thread run
- Which CPU core a thread runs on

The JVM has essentially **no control** over these low-level decisions — it delegates entirely to the OS.

### Common Scheduling Approaches (Conceptual, OS-Dependent)

Different operating systems use different scheduling algorithms internally, but common concepts include:

- **Preemptive scheduling**: The scheduler can forcibly pause a running thread (even if it hasn't voluntarily yielded) to give another thread a turn. This is standard in modern OSes like Linux, Windows, and macOS.
- **Time slicing**: Each `RUNNABLE` thread gets a small "slice" of CPU time before potentially being preempted, creating the illusion of many threads running simultaneously on a smaller number of cores.
- **Priority-based scheduling**: Threads with higher priority are generally favored for CPU time — but this is a **hint**, not a guarantee, and different OSes honor it to different degrees.

### Java's Thread Priority (A Limited Hint)

Java allows you to suggest a priority using `Thread.setPriority(int)`:

```java
Thread t = new Thread(() -> { /* work */ });
t.setPriority(Thread.MAX_PRIORITY); // 10
t.start();
```

Priority values range from `Thread.MIN_PRIORITY` (1) to `Thread.MAX_PRIORITY` (10), with `Thread.NORM_PRIORITY` (5) as the default.

**Critical caveat**: this is only a **hint** to the OS scheduler. Different operating systems map Java's 1-10 priority scale to their own native priority systems differently (or, in some cases, largely ignore it). You should **never** write code whose correctness depends on priority actually being honored a specific way — treat it purely as an optimization hint, not a control mechanism.

### Why You Cannot Rely on Execution Order

Because scheduling decisions depend on:

- The specific OS and its scheduling algorithm
- The number of available CPU cores
- Current system load (other processes competing for CPU)
- JVM implementation details

...the exact interleaving of thread execution is **fundamentally non-deterministic** from your program's perspective. Two runs of the same program, on the same machine, can produce different execution orders.

---

## Internal Working

Step-by-step of what happens when multiple threads are `RUNNABLE` at once:

1. Multiple threads are in the `RUNNABLE` state, all wanting CPU time.
2. The OS scheduler maintains an internal queue (or more complex structure) of runnable threads across the entire system (not just your JVM process — every process's threads compete together).
3. Based on its scheduling algorithm (which may consider priority, wait time, fairness policies, and more), the OS picks a thread and assigns it to an available CPU core.
4. That thread runs for a "time slice" — a small duration, often just milliseconds.
5. Either the thread voluntarily gives up the CPU (e.g., it starts waiting on I/O, calls `sleep()`, or finishes), or the OS **preempts** it once its time slice expires.
6. The scheduler picks the next thread to run, and the cycle repeats.
7. This entire process happens continuously, many times per second, invisible to your Java code — you only ever see the net effect: your threads appear to run "concurrently."

---

## Memory Diagram

```
              Operating System Scheduler
      -----------------------------------------
      |  Runnable Thread Queue (system-wide)    |
      |    - Thread A (priority 5)              |
      |    - Thread B (priority 8)              |
      |    - Thread C (priority 5)              |
      |    - [threads from OTHER processes too] |
      -----------------------------------------
                     |
                     v
       Scheduler picks next thread(s) to run
       based on OS-specific algorithm + priority hints
                     |
                     v
      -----------------------------------------
      |   CPU Core 1   |   CPU Core 2           |
      |   Thread B      |  Thread A              |
      -----------------------------------------
      (Thread C waits its turn)
```

---

## Execution Flow

```
3 threads become RUNNABLE at roughly the same time
        |
        v
OS scheduler evaluates: priority hints, wait time,
system load, available cores
        |
        v
Scheduler assigns CPU time to some subset of them
        |
        v
Time slice expires OR thread voluntarily yields/blocks
        |
        v
Scheduler re-evaluates and picks next thread(s)
        |
        v
(Repeats continuously, many times per second)
```

---

## Code Example

```java
public class ThreadSchedulerDemo {

    public static void main(String[] args) throws InterruptedException {

        Runnable countTask = () -> {
            for (int i = 0; i < 5; i++) {
                System.out.println(Thread.currentThread().getName() +
                        " (priority " + Thread.currentThread().getPriority() + ") - count: " + i);
            }
        };

        Thread low = new Thread(countTask, "Low-Priority");
        low.setPriority(Thread.MIN_PRIORITY); // 1

        Thread high = new Thread(countTask, "High-Priority");
        high.setPriority(Thread.MAX_PRIORITY); // 10

        low.start();
        high.start();

        low.join();
        high.join();

        System.out.println("Both threads finished.");
    }
}
```

### Line-by-line explanation

1. `low.setPriority(Thread.MIN_PRIORITY)` and `high.setPriority(Thread.MAX_PRIORITY)` — set explicit priority **hints** on each thread.
2. When run, you might **expect** `High-Priority` to consistently print all its output before `Low-Priority` — but in practice, on most modern operating systems, **the actual interleaving of output is often barely affected by these priority settings**, especially on multi-core machines where both threads may simply run concurrently on separate cores regardless of priority.
3. This is intentional and important: this code is designed to demonstrate that **priority is a weak hint, not a scheduling guarantee** — running this same code repeatedly, or on different machines/operating systems, can produce noticeably different (and sometimes surprising) interleaving patterns.

---

## Dry Run

Running the code above might produce output like this on one system:

```
Low-Priority (priority 1) - count: 0
High-Priority (priority 10) - count: 0
Low-Priority (priority 1) - count: 1
High-Priority (priority 10) - count: 1
...
```

Or, on a different system (or even a different run on the same system):

```
High-Priority (priority 10) - count: 0
High-Priority (priority 10) - count: 1
High-Priority (priority 10) - count: 2
Low-Priority (priority 1) - count: 0
...
```

**Both outputs are valid.** This unpredictability is exactly the point: you cannot rely on priority to guarantee any specific ordering, and your code must be correct **regardless** of how the OS chooses to interleave thread execution.

---

## Common Mistakes

1. **Using thread priority to try to enforce correctness or ordering.**
    
    - E.g., "I'll set Thread A's priority higher so it always finishes before Thread B starts." This is fundamentally unreliable — use proper synchronization tools (`join()`, `CountDownLatch`, etc.) instead, if you actually need ordering guarantees.
2. **Assuming the JVM controls scheduling.**
    
    - The JVM delegates this entirely to the OS. Different JVM implementations, on different OSes, can behave differently for identical Java code.
3. **Assuming consistent behavior across environments.**
    
    - Code that "happens to work" in a specific order during local testing may behave completely differently in production, on a different OS, or under different system load — precisely because scheduling is non-deterministic and environment-dependent.
4. **Overusing `Thread.yield()` as a scheduling control mechanism.**
    
    - `Thread.yield()` (see [[19 - Yield()]]) is merely a **hint** to the scheduler that the current thread is willing to give up its time slice — the scheduler is free to ignore this hint entirely.

---

## Production Usage

- **Real-time or latency-sensitive systems** (e.g., trading platforms) sometimes use OS-level thread affinity and priority tuning (often via native, non-Java-standard mechanisms, or OS-level tools) to influence scheduling more reliably than Java's built-in priority hints allow.
- **Most production business applications** never touch thread priority at all — they rely entirely on the default priority and trust the OS scheduler and thread pool sizing (see [[44 - Thread Pool]], [[72 - Performance Tuning]]) to achieve good throughput.
- **Container/cloud environments** (Docker, Kubernetes) add another layer of scheduling complexity: the container runtime and orchestrator also influence how much CPU time your JVM process (and thus its threads) actually receives, on top of the OS-level scheduler within the container.

---

## Interview Questions

**Beginner:**

1. Who is responsible for deciding which thread runs next in Java — the JVM or the OS?
2. What does `Thread.setPriority()` actually do?
3. Can you guarantee thread execution order using priority alone?

**Intermediate:** 4. What is preemptive scheduling, and why is it relevant to Java threads? 5. Why might the same multithreaded Java program produce different output orderings across different runs or machines? 6. What is a "time slice" in the context of thread scheduling?

**Advanced:** 7. Why does Java's thread priority scale (1-10) not map consistently across different operating systems? 8. How does running inside a container (Docker/Kubernetes) add another layer of complexity to thread scheduling behavior?

**Production-Level:** 9. If you needed more deterministic latency for a specific thread in a production system, what real options (beyond Java's built-in priority) might you consider? 10. Why should application correctness never depend on assumptions about thread scheduling order?

---

## Follow-up Questions

- Does setting all threads to `Thread.MAX_PRIORITY` make your program run faster overall? _(No — it doesn't create more CPU capacity; if all threads have equal priority, there's effectively no differentiation, and total throughput is still capped by the number of available cores.)_
- Is scheduling behavior identical across Linux, Windows, and macOS for the same Java program? _(No — each OS has its own scheduling algorithm and priority mapping, so behavior can differ even for identical Java code and JVM version.)_
- Can a single thread "starve" other threads of CPU time indefinitely, just by having high priority? _(In theory, an OS _could_ allow this under a naive priority scheme, but modern OS schedulers include fairness mechanisms specifically to prevent long-term starvation — see [[38 - Starvation]] for more detail on this general concurrency problem.)_

---

## Best Practices

- Never write code whose **correctness** depends on thread scheduling order or priority — use explicit synchronization tools (`join()`, locks, latches, barriers) instead.
- Treat `Thread.setPriority()` purely as a rare, minor optimization hint — not a control mechanism — and avoid it entirely unless you have a very specific, well-understood need.
- Design your concurrent code to be correct under **any** possible interleaving of thread execution — this is the gold standard for robust concurrent programming.
- When you need real ordering or coordination, use the tools covered later in this vault: [[20 - Join()]], [[54 - CountDownLatch]], [[55 - CyclicBarrier]], locks, and `wait()`/`notify()`.

---

## Performance Notes

- Excessive thread priority tuning rarely yields meaningful, reliable performance improvements in typical business applications — it's a low-leverage lever compared to proper thread pool sizing (see [[72 - Performance Tuning]]).
- The overhead of scheduling itself (context switches, see [[12 - Context Switching]]) becomes significant when there are far more `RUNNABLE` threads than available CPU cores — this is a much more impactful performance lever than adjusting priorities.

---

## Comparison Table

|Aspect|What Java Controls|What the OS Controls|
|---|---|---|
|Thread creation|Yes (`new Thread()`)|N/A|
|Priority hint|Yes (`setPriority()`)|Interprets/maps it (loosely)|
|Actual CPU time allocation|No|Yes|
|Which core a thread runs on|No (not directly, without native APIs)|Yes|
|Preemption timing|No|Yes|
|Execution order guarantee|No|No (non-deterministic by design)|

---

## Summary

- The **operating system**, not the JVM, is responsible for actual thread scheduling decisions.
- Java's `Thread.setPriority()` is only a **hint**, interpreted differently (or sometimes largely ignored) across different operating systems.
- Thread execution order and interleaving is **fundamentally non-deterministic** — your code must be correct regardless of scheduling behavior.
- Never rely on priority or assumed scheduling behavior for program correctness — use explicit synchronization mechanisms instead.
- Container/cloud environments add yet another layer of scheduling complexity on top of the OS.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|Thread scheduler|Lives at the OS level, not controlled by the JVM|
|`setPriority()`|A weak hint (1-10), not a guarantee|
|Time slicing|Each thread gets a small CPU turn before being preempted|
|Execution order|Non-deterministic — never rely on it for correctness|
|Cross-platform consistency|Not guaranteed — different OSes schedule differently|

---

## Keywords

Thread Scheduler, OS Scheduler, Preemptive Scheduling, Time Slicing, Thread Priority, Non-Deterministic, Starvation

---

## Related Notes

[[09 - Thread Lifecycle]] [[12 - Context Switching]] [[15 - Thread Priority]] [[19 - Yield()]] [[38 - Starvation]]

---

## Practice Exercises

**Easy:**

1. Create two threads with `MIN_PRIORITY` and `MAX_PRIORITY`, each printing 10 messages, and run the program 5 times. Record whether the output order is consistent across runs.
2. Print `Thread.currentThread().getPriority()` for the main thread — what is its default priority?

**Medium:** 3. Create 4 threads, all with default (`NORM_PRIORITY`) settings, each doing a small amount of CPU work, and observe how their output interleaves across multiple runs. 4. Research (or test) what happens if you try to set a priority value outside the range 1-10 (e.g., `setPriority(15)`).

**Hard:** 5. Write a program that creates significantly more `RUNNABLE` CPU-bound threads than your machine has cores (e.g., 50 threads on a 4-core machine), and measure total completion time versus running the same total work with exactly as many threads as cores. Observe the performance impact of excessive context switching.

---

## Mini Project

**Project: Scheduling Behavior Explorer**

Build a small console application that:

1. Creates 3 threads with different priorities (low, normal, high), each incrementing and printing a counter 20 times with a tiny artificial delay.
2. Runs this experiment 10 times in a row (in separate program executions or within one run using separate thread groups).
3. Logs the interleaving pattern observed each time, and computes basic statistics: how often did the high-priority thread finish first? How much did the ordering vary across runs?

**Goal:** Get concrete, empirical evidence (rather than just theoretical understanding) that thread priority is a weak, unreliable hint — not a scheduling guarantee.

---

## Common Interview Trap

**Trap Question:** _"If I want Thread A to definitely run before Thread B, can I just set Thread A's priority higher?"_

The trap is assuming priority provides a hard guarantee. The correct answer: **no** — priority is only a hint to the OS scheduler, and it does **not** guarantee any specific execution order, especially on multi-core systems where both threads might simply run concurrently on separate cores regardless of priority. If you genuinely need Thread B to wait for Thread A, you must use an explicit synchronization mechanism, such as `threadA.join()` before starting Thread B, or a coordination primitive like `CountDownLatch` (see [[20 - Join()]], [[54 - CountDownLatch]]).

---

## Frequently Asked Questions

**Q: Does Java guarantee fair scheduling among threads of the same priority?** A: Not explicitly — fairness depends on the underlying OS scheduler's algorithm, not on any guarantee from the JVM or Java language specification.

**Q: Can I pin a Java thread to a specific CPU core?** A: Not through standard Java APIs — this typically requires native/OS-specific tools or libraries outside the standard JDK.

**Q: Does increasing thread priority use more CPU resources overall?** A: No — priority only influences _scheduling order/preference_ among already-runnable threads; it doesn't grant a thread more total CPU capacity than exists on the machine.

**Q: Is thread scheduling deterministic in single-core systems?** A: Even on a single core, scheduling is still generally non-deterministic in practice, since modern OS schedulers use complex, often time- and load-based algorithms rather than a simple fixed order — you should never assume determinism, even on single-core hardware.

---

## Revision Checklist

- [ ] I understand that the OS, not the JVM, controls actual thread scheduling.
- [ ] I understand that `Thread.setPriority()` is a hint, not a guarantee.
- [ ] I can explain why the same program can produce different thread interleavings across runs or machines.
- [ ] I know not to rely on scheduling behavior for program correctness.
- [ ] I understand time slicing and preemptive scheduling at a conceptual level.

---

## Flashcards

**Q:** Who actually decides which thread runs next — the JVM or the OS? **A:** The OS (operating system scheduler).

**Q:** Does `Thread.setPriority()` guarantee a specific execution order? **A:** No — it's only a hint, interpreted differently across operating systems.

**Q:** What is preemptive scheduling? **A:** The OS can forcibly pause a running thread to let another thread run, even if the first thread hasn't voluntarily yielded.

**Q:** Should program correctness ever depend on thread scheduling order? **A:** No — always use explicit synchronization tools if ordering matters.

**Q:** What additional layer of scheduling complexity do containers (Docker/Kubernetes) introduce? **A:** The container runtime/orchestrator also influences how much CPU time the JVM process receives, on top of the OS-level scheduler.

---

## References

- Oracle Java Tutorials — Thread Priority: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- `java.lang.Thread` API Documentation (`setPriority`, `MIN_PRIORITY`, `MAX_PRIORITY`, `NORM_PRIORITY`)
- Operating Systems Concepts (Silberschatz et al.) — CPU Scheduling chapter (general OS reference)