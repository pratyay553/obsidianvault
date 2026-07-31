# Starvation

## Objective

By the end of this note, you will understand:

- What starvation is, and how it differs from both deadlock and livelock
- The common causes: thread priority misuse, unfair locks, and greedy threads
- How Java's default (unfair) locking mechanisms can contribute to starvation
- Practical strategies for ensuring fairness where it genuinely matters

This note completes the "three classic liveness failures" trilogy alongside [[36 - Deadlock]] and [[37 - Livelock]] — all three prevent a system from making the progress it should, but through different mechanisms.

---

## Why should I learn this?

Starvation is the subtlest of the three liveness failures — it doesn't cause a total freeze (deadlock) or an obvious CPU-burning loop (livelock); it just means **one particular thread (or a few) never gets its fair share of a shared resource**, while everyone else proceeds normally. This can manifest as a single user's requests being mysteriously, consistently slow in an otherwise healthy system — a genuinely tricky production issue to diagnose without understanding this specific failure mode.

---

## Beginner Explanation

Imagine a busy coffee shop where the barista always serves whoever asks loudest or pushes to the front, rather than respecting a first-come-first-served line. A quiet, polite customer who arrived first might end up waiting far longer than customers who arrived later but were more assertive — potentially waiting **indefinitely** if there's always someone else pushing ahead. That polite customer is being "starved" of service — not blocked in some formal, structural sense, just perpetually deprioritized in practice.

**Starvation** in Java concurrency is exactly this: a thread that's technically eligible to proceed (not deadlocked, not livelocked) but, due to scheduling decisions, priority settings, or unfair lock implementations, keeps getting passed over in favor of other threads, potentially for a very long time or indefinitely.

---

## Real World Analogy

**Hospital Analogy**

Imagine an emergency room that always treats the most severe cases first (reasonably so, for genuine emergencies) — but suppose the hospital's triage system is poorly designed and constantly receives a steady stream of moderately urgent cases, such that a patient with a **less** urgent (but still real) condition keeps getting pushed back, again and again, as new moderately-urgent patients arrive. If this pattern continues indefinitely, that lower-priority patient could theoretically wait **forever**, never quite reaching the front of the queue — not because anyone is deliberately ignoring them, but because the system's prioritization logic, in aggregate, never lets their turn come up.

---

## Technical Explanation

### Common Causes of Starvation in Java

**1. Misused Thread Priority (see [[15 - Thread Priority]])**

```java
Thread highPriority = new Thread(cpuIntensiveTask);
highPriority.setPriority(Thread.MAX_PRIORITY);

Thread lowPriority = new Thread(anotherCpuIntensiveTask);
lowPriority.setPriority(Thread.MIN_PRIORITY);
```

On some operating systems/JVM combinations, if many high-priority CPU-bound threads are constantly runnable, a low-priority thread might rarely (or never, on certain platforms) get scheduled — a form of priority-based starvation. (Recall from [[11 - Thread Scheduler]] that priority behavior is platform-dependent, which is itself part of why this risk varies.)

**2. Unfair Lock Implementations**

Java's default `synchronized` and default (non-fair) `ReentrantLock` provide **no** guarantee about the order in which waiting threads are granted access. Under sustained contention with many threads, it's theoretically possible (though often statistically rare in practice) for a specific thread to be repeatedly "unlucky" and grant-ordering to consistently favor other threads.

**3. Greedy Threads Holding Locks Too Long**

If some threads hold a shared lock for a long time (repeatedly, in a loop), other threads waiting for brief access to the same lock can experience significant, disproportionate delays — especially if the greedy threads' pattern of acquisition happens to align unfavorably with the waiting threads' timing.

### The Fix: Fair Locks

```java
Lock fairLock = new ReentrantLock(true); // fair mode
```

As covered in [[32 - ReentrantLock]], passing `true` enables **fair mode**, where the lock is granted to waiting threads roughly in the order they requested it (FIFO-like behavior) — directly addressing starvation risk, at the cost of reduced overall throughput due to the additional coordination overhead required.

### A Demonstrable Starvation Scenario

```java
class UnfairResource {
    private final Lock lock = new ReentrantLock(); // UNFAIR by default

    void useResource(String threadName) {
        lock.lock();
        try {
            System.out.println(threadName + " using the resource.");
        } finally {
            lock.unlock();
        }
    }
}
```

If many "greedy" threads constantly loop, acquiring and releasing this lock as fast as possible, a single "polite" thread that only occasionally tries to acquire it can end up waiting disproportionately long — not forever necessarily (that would edge toward deadlock-like severity), but noticeably, unfairly longer than it "should," purely due to the lack of any ordering guarantee.

---

## Internal Working

Conceptually, why unfair locks can lead to starvation:

1. An unfair lock, upon becoming free, typically grants access to **whichever waiting thread the JVM/OS scheduler happens to select next** — which is often (for performance reasons) simply the next thread that happens to be ready to try, not necessarily the one that's been waiting longest.
2. If a large number of "aggressive" threads are constantly re-attempting acquisition immediately after release, they can statistically "win" the race to re-acquire far more often than a thread that only occasionally attempts to acquire the lock.
3. Over many iterations, this can compound into a real, measurable disparity — the "quiet" thread accumulates significantly more total wait time than its "fair share" would suggest.

---

## Memory Diagram

```
        Unfair Lock (released, up for grabs)
   -----------------------------------------
   |  Waiting threads: [T1, T2, T3, ..., T50, QuietThread]  |
   -----------------------------------------
              |
              v
   Lock granted to whichever thread the JVM/OS
   happens to schedule/select next -- NOT
   necessarily in arrival order

   If T1-T50 are constantly re-attempting aggressively,
   QuietThread (which attempts far less often) may be
   "unlucky" far more often than statistically expected,
   accumulating disproportionate wait time.
```

---

## Execution Flow

```
Many "greedy" threads constantly acquire/release an unfair lock
        |
        v
A "quiet" thread occasionally also wants the lock
        |
        v
Lock release -> JVM/OS selects next thread to grant access
        |
   (no strict ordering guarantee with unfair lock)
        |
        v
Greedy threads, due to their aggressive/frequent attempts,
statistically "win" access far more often
        |
        v
Quiet thread experiences disproportionately long, unfair delays
        |
        v
FIX: use a FAIR lock, or otherwise redesign contention pattern
```

---

## Code Example

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.atomic.AtomicInteger;

public class StarvationDemo {

    static void runExperiment(boolean fair) throws InterruptedException {
        Lock lock = new ReentrantLock(fair);
        AtomicInteger quietThreadAcquisitions = new AtomicInteger(0);
        AtomicInteger greedyAcquisitions = new AtomicInteger(0);
        long endTime = System.currentTimeMillis() + 2000; // run for 2 seconds

        Runnable greedyTask = () -> {
            while (System.currentTimeMillis() < endTime) {
                lock.lock();
                try {
                    greedyAcquisitions.incrementAndGet();
                } finally {
                    lock.unlock();
                }
            }
        };

        Runnable quietTask = () -> {
            while (System.currentTimeMillis() < endTime) {
                lock.lock();
                try {
                    quietThreadAcquisitions.incrementAndGet();
                } finally {
                    lock.unlock();
                }
                try { Thread.sleep(5); } catch (InterruptedException e) { Thread.currentThread().interrupt(); } // "quiet" -- pauses between attempts
            }
        };

        Thread[] greedyThreads = new Thread[10];
        for (int i = 0; i < 10; i++) greedyThreads[i] = new Thread(greedyTask);
        Thread quietThread = new Thread(quietTask);

        for (Thread t : greedyThreads) t.start();
        quietThread.start();

        for (Thread t : greedyThreads) t.join();
        quietThread.join();

        System.out.println((fair ? "FAIR" : "UNFAIR") + " lock -- Greedy total: " +
                greedyAcquisitions.get() + ", Quiet thread total: " + quietThreadAcquisitions.get());
    }

    public static void main(String[] args) throws InterruptedException {
        runExperiment(false); // unfair (default)
        runExperiment(true);  // fair
    }
}
```

### Line-by-line explanation

1. Ten "greedy" threads constantly hammer the lock as fast as possible; one "quiet" thread acquires it, then sleeps 5ms before trying again — modeling a thread with much lower contention frequency.
2. With the **unfair** lock, the greedy threads' sheer volume of attempts typically dominates, and the quiet thread's **acquisition count relative to its attempt rate** may show a disproportionately low share compared to what pure statistical fairness would predict.
3. With the **fair** lock, acquisitions are granted more strictly in request order, generally giving the quiet thread a fairer proportion relative to how often it actually asks — though note fair mode also reduces the greedy threads' total throughput, since strict ordering adds coordination overhead.

---

## Dry Run

Example output (illustrative — exact numbers vary by machine and JVM):

```
UNFAIR lock -- Greedy total: 4128763, Quiet thread total: 342
FAIR lock -- Greedy total: 891204, Quiet thread total: 287
```

Notice: total greedy acquisitions drop significantly under fair mode (much lower overall throughput), while the quiet thread's count, relative to its much lower attempt frequency, is proportionally better represented — illustrating the fairness-vs-throughput trade-off directly.

---

## Common Mistakes

1. **Assuming Java's default locks are "basically fair" without verifying.**
    
    - Both `synchronized` and default `ReentrantLock` explicitly provide **no** fairness guarantee — assuming otherwise can lead to surprising, hard-to-diagnose disparities under heavy contention.
2. **Enabling fair mode everywhere "just in case," without considering the throughput cost.**
    
    - Fair locks have real, sometimes substantial, performance overhead — apply fairness deliberately, only where genuine starvation risk and fairness requirements justify the cost.
3. **Confusing starvation with deadlock or livelock.**
    
    - Starvation doesn't require a cycle (unlike deadlock) or constant unproductive activity from the affected thread (unlike livelock) — it's simply a persistent, unfair imbalance in how often a thread gets access to a shared resource.
4. **Not considering priority-related starvation risk when using non-default thread priorities.**
    
    - As discussed in [[15 - Thread Priority]], priority is platform-dependent, and misuse can contribute to real starvation risk on some systems, even though it's just a scheduling "hint."

---

## Production Usage

- **Rate limiters and API gateways** sometimes need to guard against a subset of clients monopolizing shared resources, effectively starving other clients — often addressed through fairness-aware queuing or per-client quotas rather than raw lock fairness.
- **Batch job schedulers** must sometimes ensure low-priority/background jobs still make **some** progress, even amid a constant stream of higher-priority work — a starvation-avoidance design concern at the application/scheduling level, not just the raw lock level.
- **Fair `ReentrantLock`** is occasionally used in specific subsystems where strict request-order fairness genuinely matters (e.g., certain queue-processing or ticketing systems), accepting the throughput trade-off deliberately.
- **Thread pool configuration** (see [[44 - Thread Pool]]) can also contribute to starvation if task submission patterns allow certain task types to be perpetually deprioritized in the internal work queue.

---

## Interview Questions

**Beginner:**

1. What is starvation, in the context of concurrent programming?
2. How does starvation differ from deadlock?
3. How does starvation differ from livelock?

**Intermediate:** 4. Does Java's default `synchronized` provide any fairness guarantee? 5. What does passing `true` to `ReentrantLock`'s constructor do? 6. What is the performance trade-off of using a fair lock?

**Advanced:** 7. Explain, with a concrete scenario, how a mix of "greedy" and "quiet" threads competing for an unfair lock can lead to measurable starvation. 8. Why might thread priority misuse contribute to starvation, and why is this risk platform-dependent?

**Production-Level:** 9. Describe a real production scenario where starvation could cause a specific subset of users/requests to experience persistently poor performance, while the overall system appears healthy. 10. How would you decide whether a specific lock in your system genuinely needs fair mode, versus accepting the default unfair behavior?

---

## Follow-up Questions

- Can starvation eventually resolve itself, given enough time, unlike a true deadlock? _(It depends — statistically, if contention patterns aren't perfectly adversarial, a starved thread might eventually get through; but there's no guarantee, and sustained, severe contention imbalances can make a thread wait far longer than acceptable for practical purposes, even if not literally "forever.")_
- Is starvation always caused by locking behavior specifically? _(No — it can also arise from thread scheduling/priority decisions, unfair queue/resource allocation logic at the application level, or any system where access to a shared, limited resource isn't guaranteed to be granted in a bounded, fair manner.)_
- Does using `AtomicInteger`/CAS-based approaches (see [[31 - CAS]]) eliminate starvation risk? _(Not entirely — under extreme, sustained contention, a CAS-based retry loop could theoretically allow some threads to "win" the race disproportionately often, though this is generally considered a much rarer practical concern than lock-based starvation.)_

---

## Best Practices

- Use fair locks specifically when you have a genuine, identified need for bounded, predictable wait times across all threads — not as a default, blanket setting.
- Monitor for unusually persistent latency in specific request types/threads in production, which can be a symptom of starvation rather than a general performance problem.
- Avoid greedy, tight-loop lock acquisition patterns where possible — introduce natural pacing (via legitimate work, not artificial delays) that reduces the aggressiveness disparity between different threads competing for the same resource.
- Be cautious with non-default thread priorities, given their platform-dependent and potentially starvation-contributing behavior (see [[15 - Thread Priority]]).

---

## Performance Notes

- Fair locks meaningfully reduce throughput compared to unfair locks, due to the additional coordination overhead required to enforce strict ordering — this is a genuine, measurable cost, not just a theoretical concern (as demonstrated in this note's dry run).
- Starvation itself, when it occurs, effectively means a subset of your system's requests/threads experience degraded "effective throughput," even while aggregate system throughput may look fine — a subtle but real production performance concern.

---

## Comparison Table

|Aspect|Deadlock|Livelock|Starvation|
|---|---|---|---|
|Requires a cycle?|Yes|Not necessarily|No|
|Affected threads' state|`BLOCKED`|`RUNNABLE` (unproductive)|`RUNNABLE`/`BLOCKED` (just persistently deprioritized)|
|Does the WHOLE system stop?|Often, for the involved threads|Often, for the involved threads|No — other threads proceed normally; only the starved thread(s) suffer|
|Typical fix|Consistent lock ordering|Randomized backoff|Fair locks, priority discipline, better scheduling design|

---

## Summary

- Starvation occurs when a thread is persistently denied access to a shared resource, even though it's not blocked in a cyclical (deadlock) or unproductively-looping (livelock) sense — other threads simply keep "winning" ahead of it.
- Common causes: misused thread priorities, unfair lock implementations under heavy contention, and greedy threads monopolizing shared resources.
- The standard fix for lock-related starvation is enabling **fair mode** on `ReentrantLock` (or equivalent fairness-aware design), accepting a real throughput trade-off in exchange for bounded, predictable wait times.
- Unlike deadlock/livelock, starvation typically affects only a subset of threads, while the rest of the system continues operating normally — making it a subtler, often user-specific production symptom.

---

## Cheat Sheet

|Symptom|Likely Issue|
|---|---|
|Whole system frozen, threads `BLOCKED` in a cycle|Deadlock|
|Whole system "busy" but no progress|Livelock|
|Most requests fine, but a specific subset persistently slow|Starvation|
|Fix: fair lock|`new ReentrantLock(true)`|

---

## Keywords

Starvation, Fair Lock, Unfair Lock, Thread Priority, Liveness Failure, FIFO Ordering

---

## Related Notes

[[36 - Deadlock]] [[37 - Livelock]] [[15 - Thread Priority]] [[32 - ReentrantLock]] [[44 - Thread Pool]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's greedy-vs-quiet thread demo with the default (unfair) lock, and note the quiet thread's acquisition count relative to its attempt frequency.
2. Re-run with a fair lock and compare.

**Medium:** 3. Experiment with different ratios of greedy to quiet threads (e.g., 2 greedy vs. 1 quiet, then 50 greedy vs. 1 quiet) and observe how the disparity changes. 4. Measure and compare total throughput (combined acquisitions across all threads) between fair and unfair modes, quantifying the fairness-vs-throughput trade-off.

**Hard:** 5. Design a scenario using thread priorities (rather than lock fairness) that demonstrates priority-related starvation risk, and discuss why the results might vary across different machines/operating systems.

---

## Mini Project

**Project: Fairness Trade-off Dashboard**

Build a small console application that:

1. Runs the greedy-vs-quiet thread experiment from this note under both fair and unfair lock modes, across several different greedy-thread-count configurations (5, 20, 50).
2. Records, for each configuration and mode: total combined throughput, and the quiet thread's acquisition count.
3. Prints a summary table showing the trade-off: as fairness increases protection for the quiet thread, how much does total throughput decrease?

**Goal:** Build a data-driven, empirical understanding of the fairness-vs-throughput trade-off, equipping you to make informed, justified decisions about when fair locks are actually worth their cost in a real system.

---

## Common Interview Trap

**Trap Question:** _"If a thread is waiting a very long time to acquire a lock, but the system overall isn't frozen and other threads are making progress fine, is this a deadlock?"_

The trap is applying the "something's stuck" intuition to the wrong failure mode. The correct answer: **no** — a deadlock, by definition, requires a **circular wait** where the involved threads can never proceed, and it typically affects a whole, identifiable set of directly-interacting threads simultaneously. A single thread experiencing persistently poor lock-acquisition luck, while the rest of the system operates normally, is the signature of **starvation**, not deadlock — there's no cycle here, just an unfair (though not fundamentally deadlocked) distribution of access opportunities. The fix is also completely different: deadlock requires breaking a circular dependency (e.g., via consistent lock ordering), while starvation is addressed through fairness mechanisms (like a fair `ReentrantLock`) or redesigning the contention pattern — recognizing which failure mode you're actually facing is essential to applying the correct fix.

---

## Frequently Asked Questions

**Q: Does using `synchronized` make starvation more or less likely compared to `ReentrantLock`?** A: Neither `synchronized` nor default (unfair) `ReentrantLock` provides fairness guarantees, so both carry similar theoretical starvation risk under adversarial contention patterns; `ReentrantLock`'s advantage is that it offers an explicit, opt-in fair mode as a remedy, which `synchronized` cannot provide at all.

**Q: Is starvation always a "bug," or can some starvation risk be an acceptable trade-off?** A: In many real systems, some theoretical starvation risk is accepted as a reasonable trade-off for better average-case throughput (the common case with unfair locks) — it becomes a genuine problem specifically when it causes unacceptable, user-visible impact for the affected thread(s)/requests.

**Q: Can starvation occur even without explicit locks, purely from thread scheduling?** A: Yes — as discussed, misused thread priorities or generally adversarial CPU-bound scheduling conditions can, on some platforms, lead to a low-priority thread rarely getting CPU time at all, independent of any explicit lock contention.

**Q: Is there a way to detect starvation programmatically, similar to `jstack`'s deadlock detection?** A: Not as a built-in, automatic JVM feature — detecting starvation typically requires application-level monitoring (e.g., tracking and alerting on per-thread or per-request wait-time distributions) rather than a single built-in diagnostic command.

---

## Revision Checklist

- [ ] I can define starvation and distinguish it clearly from both deadlock and livelock.
- [ ] I understand that Java's default locking mechanisms provide no fairness guarantee.
- [ ] I know how to enable fair mode on `ReentrantLock`, and understand its throughput cost.
- [ ] I can describe at least one real production scenario where starvation would manifest as a subtle, subset-specific performance issue.
- [ ] I understand that starvation, unlike deadlock/livelock, typically leaves the rest of the system functioning normally.

---

## Flashcards

**Q:** What is starvation? **A:** A situation where a thread is persistently denied access to a shared resource, even though it's not blocked in a cycle (deadlock) or looping unproductively (livelock).

**Q:** Does Java's default `synchronized` provide fairness guarantees? **A:** No.

**Q:** How do you enable fair mode on a `ReentrantLock`? **A:** Pass `true` to its constructor: `new ReentrantLock(true)`.

**Q:** What's the trade-off of using a fair lock? **A:** Reduced overall throughput, due to the added coordination overhead needed to enforce strict acquisition ordering.

**Q:** Does starvation typically affect the whole system, or a subset of threads? **A:** Typically just a subset — the rest of the system usually continues operating normally.

---

## References

- Oracle Java Tutorials — Starvation and Livelock: https://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html
- `java.util.concurrent.locks.ReentrantLock` API Documentation (fairness constructor parameter)