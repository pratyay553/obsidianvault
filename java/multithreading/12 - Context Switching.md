# Context Switching

## Objective

By the end of this note, you will understand:

- What a context switch actually is, at the CPU/OS level
- Why context switching has a real, measurable performance cost
- How to recognize excessive context switching as a production performance problem
- Why "just add more threads" is not a free performance lever

This note builds on [[11 - Thread Scheduler]] and connects directly to [[72 - Performance Tuning]].

---

## Why should I learn this?

Context switching is the **hidden cost** of concurrency that beginners rarely think about. It's easy to assume that adding more threads is always beneficial — but every time the CPU switches from running one thread to running another, there's real overhead involved. Too many threads competing for too few CPU cores can actually make your application **slower**, not faster.

Understanding context switching is essential for:

- Correctly sizing thread pools in production (see [[44 - Thread Pool]])
- Explaining, in interviews, why "more threads = more speed" is a myth for CPU-bound work
- Diagnosing real production performance problems where CPU usage is high but actual useful work throughput is low

---

## Beginner Explanation

Imagine you're studying for 3 different exams at once, switching between them every 5 minutes: 5 minutes on Math, then 5 minutes on History, then 5 minutes on Science, then back to Math, and so on.

Every time you switch subjects, there's a "mental reload" cost — you have to remember where you left off, re-read your last few notes, and refocus. If you switch too often, you spend more time **reloading your mental context** than actually studying.

This is exactly what a **context switch** is for a CPU:

- The CPU can only truly execute one thread's instructions at a time per core.
- When it switches from running Thread A to running Thread B, it must **save** Thread A's current state (its "place in the book") and **load** Thread B's saved state.
- This save-and-load operation takes real time — time that isn't spent doing useful work for either thread.

---

## Real World Analogy

**Restaurant Kitchen Analogy**

Imagine a chef cooking 3 dishes at once, switching between them every 30 seconds instead of finishing one dish before starting the next:

- Every time the chef switches from Dish A to Dish B, they have to **put down** their current tool, **look at** where Dish B was left off, and **pick up** the right tool/ingredients for Dish B.
- This switching overhead doesn't directly cook any food — it's pure "overhead" time.
- If the chef switches too frequently (say, every 2 seconds instead of 30), they'll spend almost all their time switching and very little time actually cooking — total output plummets, even though the chef is technically "very busy."

This is exactly the risk with too many threads: the CPU spends more time switching between them than executing their actual instructions.

---

## Technical Explanation

### What a Context Switch Actually Involves

When the OS scheduler decides to stop running one thread and start running another on a given CPU core, it must:

1. **Save the current thread's state** (called its "context"):
    - CPU registers (including the program counter — where execution should resume)
    - Stack pointer
    - Other CPU-specific state
2. **Store this saved state** somewhere associated with that thread (managed by the OS kernel).
3. **Load the next thread's previously saved state** into the CPU's registers.
4. **Resume execution** of the next thread exactly where it left off.

This entire save/load sequence takes real CPU cycles — typically measured in **microseconds**, but at high thread counts and high switching frequency, this adds up to meaningful overhead.

### Why Context Switches Also Hurt CPU Cache Performance

Beyond the direct save/load cost, there's a subtler, often larger cost: **CPU cache invalidation**.

- Modern CPUs rely heavily on fast on-chip caches (L1, L2, L3) to avoid slow main-memory access.
- While Thread A runs, the cache fills up with data and instructions relevant to Thread A.
- When the CPU switches to Thread B, Thread B's relevant data is likely **not** in the cache yet (a "cold cache"), forcing slower main-memory fetches until the cache "warms up" again for Thread B.
- If threads switch very frequently, the cache never gets a chance to stay "warm" for any one thread, degrading overall performance significantly — often more than the raw register save/load cost itself.

### Voluntary vs. Involuntary Context Switches

- **Voluntary context switch**: A thread willingly gives up the CPU — e.g., it calls `sleep()`, blocks on I/O, or calls `wait()`. It's not being forced off; it has nothing to do right now anyway.
- **Involuntary context switch**: The OS scheduler **forcibly** preempts a running thread (even though it wants to keep running) because its time slice has expired, or a higher-priority thread needs to run. This is a normal part of preemptive scheduling (see [[11 - Thread Scheduler]]).

### Thread Count vs. Core Count

- If you have **fewer or equal** `RUNNABLE` threads than CPU cores, in principle every thread could run continuously without needing to context switch with other application threads (though the OS still schedules other system/process threads too).
- If you have **far more** `RUNNABLE` threads than cores, the OS must constantly switch between them to give each a fair turn — and the more threads you add beyond the core count, the more switching overhead accumulates, often with diminishing (or negative) returns on actual throughput.

---

## Internal Working

Step-by-step of what happens during a context switch:

1. The OS scheduler decides Thread A (currently running on Core 1) must stop — either voluntarily (it's blocking on something) or involuntarily (time slice expired, or higher priority thread needs to run).
2. The CPU's current register values (including the program counter marking exactly where Thread A was executing) are copied out and saved into Thread A's kernel-managed thread control block.
3. The scheduler selects the next thread to run — say, Thread B.
4. Thread B's previously saved register values are loaded back into the CPU's registers.
5. The CPU resumes execution starting exactly where Thread B last left off.
6. Thread A remains paused (in `RUNNABLE`, waiting for its next turn, or in `BLOCKED`/`WAITING` if it gave up the CPU voluntarily for a specific reason) until the scheduler selects it again.
7. Meanwhile, CPU caches gradually "forget" Thread A's data and start filling with Thread B's data, incurring the cache-related slowdown described above.

---

## Memory Diagram

```
     Time -->

     Core 1 timeline:
     [--- Thread A running ---][SWITCH][--- Thread B running ---][SWITCH][--- Thread A running ---]
                                  ^                                  ^
                          Save A's registers                Save B's registers
                          Load B's registers                Load A's registers
                          (small time cost,                 (small time cost,
                           cache also starts                 cache starts
                           "cooling" for A)                  "cooling" for B)
```

---

## Execution Flow

```
Thread A running on Core 1
        |
        v
Time slice expires (or Thread A blocks on I/O)
        |
        v
OS saves Thread A's register state
        |
        v
OS loads Thread B's previously saved register state
        |
        v
Thread B resumes execution on Core 1
        |
        v
(CPU cache gradually shifts from "warm for A" to "warm for B")
        |
        v
Eventually, Thread B's time slice expires / it blocks
        |
        v
(cycle repeats, potentially switching back to Thread A or another thread)
```

---

## Code Example

While you cannot directly observe register-level context switches from within Java code, you can demonstrate the **performance impact** of excessive context switching with a simple benchmark comparing "reasonable" versus "excessive" thread counts for CPU-bound work:

```java
public class ContextSwitchingDemo {

    static long busyWork(long iterations) {
        long sum = 0;
        for (long i = 0; i < iterations; i++) {
            sum += i;
        }
        return sum;
    }

    static void runWithThreadCount(int threadCount, long iterationsPerThread) throws InterruptedException {
        Thread[] threads = new Thread[threadCount];

        long start = System.nanoTime();

        for (int i = 0; i < threadCount; i++) {
            threads[i] = new Thread(() -> busyWork(iterationsPerThread));
            threads[i].start();
        }

        for (Thread t : threads) {
            t.join();
        }

        long end = System.nanoTime();
        System.out.printf("Threads: %-5d Time: %.2f ms%n", threadCount, (end - start) / 1_000_000.0);
    }

    public static void main(String[] args) throws InterruptedException {
        int availableCores = Runtime.getRuntime().availableProcessors();
        System.out.println("Available CPU cores: " + availableCores);

        long totalIterations = 2_000_000_000L; // fixed total amount of work

        // Try increasing thread counts, keeping TOTAL work constant
        for (int threadCount : new int[]{1, availableCores, availableCores * 4, availableCores * 20}) {
            runWithThreadCount(threadCount, totalIterations / threadCount);
        }
    }
}
```

### Line-by-line explanation

1. `busyWork(iterations)` — a pure CPU-bound task (no I/O, no sleeping) that just does arithmetic in a loop.
2. `runWithThreadCount(...)` — divides a **fixed total amount of work** across a given number of threads, and measures total wall-clock time.
3. The `main()` method tests several thread counts: 1 thread, exactly as many threads as CPU cores, 4x the core count, and 20x the core count — all doing the **same total amount of work**, just divided differently.
4. **Expected observation**: performance improves as thread count increases up to roughly the number of available cores, then **plateaus or even degrades** as thread count grows far beyond the core count, due to context-switching overhead and cache thrashing — directly demonstrating the concept from this note.

---

## Dry Run

Example results on a hypothetical 4-core machine (actual numbers vary by machine):

|Thread Count|Approx. Time|
|---|---|
|1|2000 ms|
|4 (= cores)|550 ms|
|16 (4x cores)|620 ms|
|80 (20x cores)|950 ms|

Notice: going from 1 thread to 4 threads (matching the core count) gives a large speedup. But going from 4 to 16 to 80 threads, despite doing the exact same total amount of work, actually gets **slower** — this is context-switching and cache-thrashing overhead outweighing any benefit, since there's no more parallel capacity to gain beyond the core count for pure CPU-bound work.

---

## Common Mistakes

1. **Assuming more threads always means better performance.**
    
    - As shown above, for CPU-bound work, performance typically plateaus (or worsens) once thread count significantly exceeds the number of available cores.
2. **Sizing thread pools arbitrarily large "just to be safe."**
    
    - A thread pool with hundreds of threads on a machine with only a handful of cores, running CPU-bound tasks, will suffer from excessive context switching rather than benefiting from "extra parallelism" that doesn't actually exist at the hardware level.
3. **Ignoring the difference between CPU-bound and I/O-bound thread pool sizing.**
    
    - I/O-bound thread pools can often benefit from more threads than cores (since those threads spend most of their time waiting, not computing — see [[03 - Why Multithreading Exists]]), but CPU-bound pools should generally be sized close to the core count.
4. **Not measuring before optimizing.**
    
    - Developers sometimes tune thread counts based on intuition alone. The correct approach is to **benchmark** with realistic workloads (like the example above) to find the actual sweet spot for a given machine and workload type.

---

## Production Usage

- **Thread pool sizing formulas**: A common starting heuristic for CPU-bound thread pools is `number of CPU cores` (or `cores + 1`); for I/O-bound pools, formulas like `cores * (1 + wait_time/compute_time)` are often used — full detail in [[72 - Performance Tuning]].
- **High-frequency trading and low-latency systems** go to extreme lengths to minimize context switching, sometimes pinning specific threads to specific CPU cores (via native/OS-level tools) to avoid any unnecessary switching at all.
- **Container CPU limits** (e.g., Kubernetes CPU quotas) effectively reduce the "available core count" a JVM sees, meaning a thread pool sized for a full machine's cores can become badly oversized (and thus suffer excessive context switching) once deployed in a constrained container.
- **Monitoring tools** (e.g., `vmstat` on Linux, or JVM-level metrics) can report context switch rates, which experienced engineers watch as a signal of over-threading in production systems.

---

## Interview Questions

**Beginner:**

1. What is a context switch?
2. Why does context switching have a performance cost?
3. What's the difference between a voluntary and an involuntary context switch?

**Intermediate:** 4. Why might adding more threads than CPU cores actually slow down a CPU-bound program? 5. How does context switching affect CPU cache performance, beyond just the register save/load cost? 6. What's a reasonable starting point for sizing a thread pool for CPU-bound work?

**Advanced:** 7. How would you empirically determine the optimal thread pool size for a given CPU-bound workload on a specific machine? 8. Why do I/O-bound workloads tolerate (and often benefit from) many more threads than CPU cores, compared to CPU-bound workloads?

**Production-Level:** 9. How does running inside a container with CPU limits affect the "ideal" thread pool size, and why is this a common production misconfiguration? 10. What tools or metrics would you use to detect that a production system is suffering from excessive context switching?

---

## Follow-up Questions

- Does context switching only happen between threads of the same application? _(No — the OS scheduler manages threads across **all** processes running on the machine, not just your JVM; your threads are also competing with threads from completely unrelated applications and system processes.)_
- Is context switching entirely avoidable? _(No — some context switching is a normal, necessary part of any multitasking OS; the goal is to avoid _excessive_ switching caused by having far more runnable threads than the hardware can usefully support at once.)_
- Does I/O-bound work cause the same kind of context-switching overhead concerns as CPU-bound work? _(Less so — when a thread blocks on I/O, it voluntarily gives up the CPU, freeing it for other useful work; the "problem" scenario is specifically many CPU-bound threads competing for too few cores.)_

---

## Best Practices

- Size CPU-bound thread pools close to the number of available CPU cores (see [[72 - Performance Tuning]] for detailed formulas).
- Size I/O-bound thread pools based on how much time tasks spend waiting versus computing — these can reasonably exceed the core count.
- Always benchmark realistic workloads on the actual target environment (including container CPU limits) rather than guessing thread pool sizes.
- Avoid creating large numbers of raw, unpooled threads for CPU-bound work — this is one of the fastest ways to introduce excessive context-switching overhead in production.

---

## Performance Notes

- A single context switch typically costs on the order of low microseconds directly, but the associated cache-warming penalty can add significantly more indirect cost, depending on working set size and cache architecture.
- Systems with many more `RUNNABLE` CPU-bound threads than cores can spend a surprisingly large fraction of total CPU time on scheduling overhead rather than actual application work — this is sometimes visible in monitoring as high "system" CPU time versus "user" CPU time.
- Container CPU quotas (cgroups) can cause **throttling**, which interacts with context-switching concerns in subtle ways — a thread pool sized for "8 cores" in a container actually limited to "2 cores" worth of CPU time will behave very differently than expected.

---

## Comparison Table

|Thread Count vs. Core Count|Effect on CPU-bound Work|
|---|---|
|Threads ≤ Cores|Good — each thread can run mostly uninterrupted|
|Threads slightly > Cores|Mild overhead, often still acceptable|
|Threads >> Cores|Significant context-switching and cache-thrashing overhead; throughput plateaus or degrades|

---

## Summary

- A context switch is when the CPU stops executing one thread and starts executing another, requiring the OS to save and restore each thread's state.
- Context switching has a real cost: direct register save/load time, plus indirect CPU cache "cooling/warming" costs.
- For CPU-bound work, having far more threads than CPU cores leads to excessive context switching, which can degrade — not improve — total throughput.
- I/O-bound work tolerates many more threads than cores, since those threads spend most of their time waiting, not competing for CPU cycles.
- Thread pool sizing should be based on workload type (CPU-bound vs. I/O-bound) and should ideally be benchmarked, not guessed.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|Context switch|CPU stops one thread, starts another; saves/restores state|
|Voluntary switch|Thread gives up CPU on its own (I/O, sleep, wait)|
|Involuntary switch|OS forcibly preempts a running thread|
|Cache impact|Switching "cools" the CPU cache for the paused thread|
|CPU-bound pool sizing|~ number of CPU cores|
|I/O-bound pool sizing|Can exceed core count significantly|

---

## Keywords

Context Switch, CPU Cache, Cache Thrashing, Voluntary Switch, Involuntary Switch, Time Slice, Thread Pool Sizing, cgroups

---

## Related Notes

[[11 - Thread Scheduler]] [[44 - Thread Pool]] [[66 - CPU Cache]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Print `Runtime.getRuntime().availableProcessors()` to find out how many cores your machine reports.
2. Run a simple CPU-bound loop using 1 thread and time it; then split the same total work across `availableProcessors()` threads and compare timings.

**Medium:** 3. Reproduce the benchmark from this note's Code Example on your own machine, and record the actual results for your specific core count. 4. Modify the benchmark to test thread counts between the core count and 4x the core count in smaller increments (e.g., cores, cores+2, cores+4, cores*2) to find where the plateau/degradation begins on your machine.

**Hard:** 5. Research and, if possible, use an OS tool (like `vmstat` on Linux, or Task Manager's performance counters on Windows) to observe actual context switch rates while running your benchmark with a very high thread count.

---

## Mini Project

**Project: Thread Pool Sizing Benchmark Tool**

Build a small console application that:

1. Accepts a CPU-bound workload (e.g., computing primes up to N) and a fixed total amount of work.
2. Runs this workload across a range of thread counts (1, cores/2, cores, cores_2, cores_4, cores*10).
3. Times each configuration and prints a simple table/chart (even ASCII bar chart) showing time versus thread count.
4. Highlights which thread count achieved the best performance.

**Goal:** Build a reusable tool that empirically demonstrates (and lets you verify on your own hardware) exactly where the "diminishing returns" point is for CPU-bound thread pool sizing — turning an abstract concept into hands-on, measured evidence.

---

## Common Interview Trap

**Trap Question:** _"If my server has 4 CPU cores, and I create a thread pool of 4 threads for CPU-bound work, will it be exactly 4x faster than a single thread?"_

The trap is assuming a perfectly linear speedup. In practice, you'll typically see **less than a perfect 4x speedup**, even with exactly 4 threads on 4 cores, because:

- The OS itself, and other system/background processes, also need some CPU time, so not 100% of all 4 cores is purely available to your 4 application threads.
- There's still some context-switching and coordination overhead (e.g., if your threads need to synchronize on shared state at all).
- Real hardware effects (memory bandwidth contention, cache behavior) mean theoretical maximum speedup is rarely achieved exactly in practice.

The correct, nuanced answer acknowledges that near-linear speedup up to the core count is a reasonable **expectation**, but a perfect linear multiplier is an idealization, not a guarantee.

---

## Frequently Asked Questions

**Q: Does context switching only affect performance, or can it also cause correctness bugs?** A: Context switching itself doesn't directly cause correctness bugs — but it's precisely the unpredictable timing of context switches that makes race conditions (see [[26 - Race Condition]]) manifest inconsistently, since the exact interleaving of operations from different threads depends on when switches happen.

**Q: Is context switching unique to Java, or is it a general OS concept?** A: It's a fundamental, general operating system concept that applies to all multithreaded/multiprocess programs on any language or platform — Java threads are simply OS threads under the hood, so they're subject to exactly the same OS-level scheduling and context-switching behavior.

**Q: Can virtual threads (Project Loom, see [[78 - Virtual Threads (Project Loom)]]) avoid context-switching overhead?** A: Virtual threads change the picture significantly for I/O-bound workloads by multiplexing many virtual threads onto far fewer OS "carrier" threads, reducing the number of expensive OS-level context switches needed — though CPU-bound work is still ultimately bound by the number of real CPU cores available.

**Q: How can I reduce context-switching overhead in my own application?** A: Primarily by sizing thread pools appropriately for your workload type (see [[72 - Performance Tuning]]) rather than over-provisioning threads, and by minimizing unnecessary blocking/synchronization that causes threads to frequently switch between running and waiting states.

---

## Revision Checklist

- [ ] I can explain what a context switch is and why it has a real performance cost.
- [ ] I understand the difference between voluntary and involuntary context switches.
- [ ] I understand why context switching also affects CPU cache performance, not just raw switching time.
- [ ] I can explain why more threads than CPU cores can hurt CPU-bound performance.
- [ ] I understand why I/O-bound workloads tolerate higher thread counts than CPU-bound workloads.

---

## Flashcards

**Q:** What is a context switch? **A:** The process of the CPU saving one thread's state and loading another thread's state to switch which thread is executing.

**Q:** What's the difference between a voluntary and involuntary context switch? **A:** Voluntary: the thread gives up the CPU itself (e.g., blocking on I/O). Involuntary: the OS forcibly preempts the thread.

**Q:** Besides the direct save/load cost, what other performance cost does context switching cause? **A:** CPU cache "cooling" for the paused thread and "warming up" for the new thread, causing slower memory access until the cache adjusts.

**Q:** For CPU-bound work, what's a reasonable thread pool size relative to CPU cores? **A:** Roughly equal to the number of available CPU cores.

**Q:** Why can I/O-bound thread pools reasonably exceed the CPU core count? **A:** Because those threads spend most of their time waiting (not competing for CPU cycles), so more of them can be "in flight" without causing excessive context-switching overhead.

---

## References

- Oracle Java Tutorials — Concurrency: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- Operating Systems Concepts (Silberschatz et al.) — CPU Scheduling and Context Switching chapters
- `java.lang.Runtime#availableProcessors()` API Documentation