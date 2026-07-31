# Livelock

## Objective

By the end of this note, you will understand:

- What a livelock is, and precisely how it differs from a deadlock
- Why livelocks are, in some ways, harder to detect than deadlocks
- A classic, reproducible livelock scenario ("overly polite threads")
- How to design retry/backoff logic that avoids introducing livelock

This note builds directly on [[36 - Deadlock]] — livelock is a closely related but distinct failure mode, often confused with deadlock in interviews.

---

## Why should I learn this?

Livelock is a sneaky bug because it doesn't look "stuck" the way a deadlock does — CPU usage stays high, threads are clearly "doing something," yet the system makes **zero actual forward progress**. Engineers who only know to check "are all threads blocked?" (the deadlock signature) can be completely misled by a livelock, where threads are very much **not** blocked — they're actively, endlessly reacting to each other.

---

## Beginner Explanation

Imagine two people approaching each other in a narrow hallway. Both politely try to step aside to let the other pass — but they **both** step to their own left at the same time, ending up blocking each other again. Both apologize and both step to their own right this time — and end up blocking each other again. This could, in principle, continue **forever**: both people are constantly moving, constantly "doing something" (unlike a true deadlock where both would just stand frozen), but neither ever actually gets past the other.

This is exactly a **livelock**: threads that are actively responding to each other's state changes, but in a way that never actually leads to progress — they're busy, but stuck.

---

## Real World Analogy

**Traffic Signal Analogy**

Imagine two cars arriving at an unmarked 4-way intersection at the same moment, each politely waiting for the other to go first. After a pause, both drivers, being polite, wave the other one through **at the exact same time** — so both proceed simultaneously, nearly colliding, both slam on the brakes, and now both wait again... and both wave the other through again at the same time... This "polite deadlock" between two drivers, if it repeats indefinitely with perfectly mirrored timing, is a real-world livelock: constant activity (waving, braking, waiting), zero actual progress (neither car ever gets through).

---

## Technical Explanation

### The Defining Characteristic: Active but Unproductive

The key technical distinction from deadlock:

|Aspect|Deadlock|Livelock|
|---|---|---|
|Thread state|`BLOCKED` (or `WAITING`) — fully suspended|`RUNNABLE` — actively executing|
|CPU usage|None (from the stuck threads)|Can be high — threads are busy "reacting"|
|Detectability via thread dump|Explicit deadlock detection possible|Much harder — threads just look like they're doing normal work, repeatedly|

### A Classic Livelock Scenario: The "Overly Polite" Retry Pattern

```java
class Resource {
    private volatile boolean inUse = false;
}

void tryUseResources(Resource myResource, Resource otherResource) {
    while (true) {
        if (!myResource.inUse) {
            myResource.inUse = true;
            if (!otherResource.inUse) {
                // do the work
                myResource.inUse = false;
                return;
            } else {
                // "polite" behavior: back off and let the other one go
                myResource.inUse = false;
            }
        }
        // retry immediately -- THIS is the livelock risk
    }
}
```

If two threads run this same logic against each other's resources **in lockstep** — each seeing the other's resource as "in use" at the same moment, each politely backing off, then immediately retrying at the same moment again — they can loop **forever**, constantly retrying, constantly backing off for each other, never actually both succeeding at the same time.

### The Standard Fix: Randomized Backoff

```java
import java.util.concurrent.ThreadLocalRandom;

void tryUseResourcesFixed(Resource myResource, Resource otherResource) throws InterruptedException {
    while (true) {
        if (!myResource.inUse) {
            myResource.inUse = true;
            if (!otherResource.inUse) {
                // do the work
                myResource.inUse = false;
                return;
            } else {
                myResource.inUse = false;
            }
        }
        // Add RANDOM delay before retrying -- breaks the lockstep pattern
        Thread.sleep(ThreadLocalRandom.current().nextInt(10, 50));
    }
}
```

By introducing a small, **random** delay before retrying, the two threads' retry timings quickly desynchronize, breaking the perfectly mirrored lockstep pattern that caused the livelock — eventually, one thread will retry at a moment when the other isn't also retrying, and progress is made.

---

## Internal Working

Step-by-step of how a livelock forms in the "overly polite" scenario:

1. Thread A marks its own resource as in-use, checks Thread B's resource — sees it's in-use (Thread B just marked it), so Thread A politely backs off, releasing its own resource.
2. **At the same time**, Thread B marks its own resource as in-use, checks Thread A's resource — sees it's in-use (Thread A just marked it), so Thread B also backs off.
3. Both threads immediately retry (no delay), and — due to consistent timing (e.g., both running similar, fast code paths with no variability) — end up repeating **exactly** the same sequence: both mark in-use, both see the other as busy, both back off, both retry immediately.
4. This can repeat indefinitely, with both threads fully `RUNNABLE` and consuming CPU the entire time, yet neither ever completing its actual work.

---

## Memory Diagram

```
   Time step 1:
   Thread A: mine=true, checks B: true (busy) -> backs off, mine=false
   Thread B: mine=true, checks A: true (busy) -> backs off, mine=false
   (mirrored, no progress)

   Time step 2 (immediate retry, same pattern):
   Thread A: mine=true, checks B: true -> backs off
   Thread B: mine=true, checks A: true -> backs off
   (mirrored again... and again... forever)
```

---

## Execution Flow

```
Thread A loop: mark busy -> check B -> B busy? -> back off -> IMMEDIATE retry
Thread B loop: mark busy -> check A -> A busy? -> back off -> IMMEDIATE retry
        |
        v
Both loops execute in near-perfect lockstep, repeating the
exact same "polite backoff" pattern indefinitely
        |
        v
CPU usage: HIGH (both threads constantly looping)
Progress: ZERO (neither ever completes)
        |
        v
FIX: introduce RANDOM delay before retry, breaking the lockstep
```

---

## Code Example

```java
import java.util.concurrent.ThreadLocalRandom;
import java.util.concurrent.atomic.AtomicInteger;

public class LivelockDemo {

    static class Resource {
        volatile boolean inUse = false;
    }

    static AtomicInteger attemptsWithoutBackoff = new AtomicInteger(0);

    static boolean tryOnce(Resource mine, Resource theirs) {
        if (!mine.inUse) {
            mine.inUse = true;
            if (!theirs.inUse) {
                mine.inUse = false;
                return true; // success!
            } else {
                mine.inUse = false; // politely back off
            }
        }
        return false;
    }

    public static void main(String[] args) throws InterruptedException {

        Resource resourceA = new Resource();
        Resource resourceB = new Resource();

        // Demonstrate the livelock risk with a bounded attempt count (to avoid an ACTUAL infinite loop in this demo)
        Thread threadUsingA = new Thread(() -> {
            int attempts = 0;
            while (!tryOnce(resourceA, resourceB) && attempts < 1_000_000) {
                attempts++;
                // NO backoff here -- this is the livelock-prone version
            }
            System.out.println("Thread using A finished after " + attempts + " attempts (no backoff).");
        });

        Thread threadUsingB = new Thread(() -> {
            int attempts = 0;
            while (!tryOnce(resourceB, resourceA) && attempts < 1_000_000) {
                attempts++;
            }
            System.out.println("Thread using B finished after " + attempts + " attempts (no backoff).");
        });

        threadUsingA.start();
        threadUsingB.start();
        threadUsingA.join();
        threadUsingB.join();

        System.out.println("--- Now with randomized backoff ---");

        Thread fixedA = new Thread(() -> {
            int attempts = 0;
            try {
                while (!tryOnce(resourceA, resourceB)) {
                    attempts++;
                    Thread.sleep(ThreadLocalRandom.current().nextInt(1, 5));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Fixed thread A finished after " + attempts + " attempts (with backoff).");
        });

        Thread fixedB = new Thread(() -> {
            int attempts = 0;
            try {
                while (!tryOnce(resourceB, resourceA)) {
                    attempts++;
                    Thread.sleep(ThreadLocalRandom.current().nextInt(1, 5));
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Fixed thread B finished after " + attempts + " attempts (with backoff).");
        });

        fixedA.start();
        fixedB.start();
        fixedA.join();
        fixedB.join();
    }
}
```

### Line-by-line explanation

1. The first pair of threads (`threadUsingA`, `threadUsingB`) retries **immediately** with no delay — under the right (unlucky) timing conditions on some systems, this can approach livelock-like behavior, requiring many attempts (bounded here at 1,000,000 purely so this demo doesn't hang forever for teaching purposes).
2. The second pair (`fixedA`, `fixedB`) adds a small **random** delay before each retry — this typically resolves the contention far faster, since the random timing quickly breaks any lockstep pattern.
3. **Note**: true livelock is timing-sensitive and not always perfectly reproducible on every machine/JVM — this demo is designed to illustrate the _mechanism_ and the _fix_, even if the unbounded version doesn't hang forever on every single run (system-level jitter often introduces enough natural randomness to eventually break the pattern) — this is itself part of the lesson about how subtle and environment-dependent this class of bug can be.

---

## Dry Run

**Without backoff (illustrative — actual attempt counts vary by machine):**

```
Thread using A finished after 47281 attempts (no backoff).
Thread using B finished after 52103 attempts (no backoff).
```

**With randomized backoff (typically far fewer attempts needed):**

```
Fixed thread A finished after 3 attempts (with backoff).
Fixed thread B finished after 2 attempts (with backoff).
```

---

## Common Mistakes

1. **Implementing "polite" retry/backoff logic without any randomization.**
    
    - Deterministic, uniform backoff (e.g., "always wait exactly 10ms before retrying") can still result in synchronized retry patterns across multiple threads, especially if they started at similar times — randomization is essential to reliably break lockstep.
2. **Assuming livelock is impossible if no threads are ever `BLOCKED`.**
    
    - This is exactly the trap — livelocked threads are `RUNNABLE` the entire time, actively consuming CPU, which is precisely why checking only for `BLOCKED` threads (the deadlock signature) misses livelock entirely.
3. **Debugging a suspected livelock the same way as a deadlock (checking `jstack` for a deadlock cycle).**
    
    - `jstack`'s automatic deadlock detection won't find a livelock, since no thread is actually `BLOCKED` waiting on a lock — you'd need to observe **repeated, unproductive activity** across multiple thread dumps taken over time (see [[10 - Thread States]]) to recognize the pattern instead.
4. **Over-engineering collision-avoidance logic that's "too polite," causing exactly this failure mode.**
    
    - Ironically, code specifically designed to be considerate/cooperative (backing off for others) is exactly the kind of code most at risk of livelock if not carefully randomized.

---

## Production Usage

- **Network protocol collision handling** (e.g., Ethernet's classic collision-detection/backoff mechanisms) is the textbook real-world domain where randomized exponential backoff was popularized specifically to prevent this exact livelock-like "both retry at the same instant" problem.
- **Distributed system retry logic** (e.g., two services repeatedly backing off for each other in a resource-contention scenario) can experience application-level livelock if backoff isn't randomized (this is sometimes called the "thundering herd" problem in related contexts).
- **Optimistic concurrency control** (e.g., "compare-and-swap, retry on failure" patterns — see [[31 - CAS]]) can theoretically suffer livelock-like behavior under extremely high, perfectly synchronized contention, though real-world CPU/OS-level timing jitter usually provides enough natural randomness to avoid true, permanent livelock in practice.

---

## Interview Questions

**Beginner:**

1. What is a livelock?
2. How does livelock differ from deadlock in terms of thread state and CPU usage?
3. Give a real-world, non-code analogy for a livelock.

**Intermediate:** 4. Why is livelock generally harder to detect than deadlock using standard tools like `jstack`? 5. What is the standard fix for preventing livelock in retry-based logic? 6. Why does deterministic (non-random) backoff sometimes fail to prevent livelock?

**Advanced:** 7. Explain, step by step, how two "overly polite" threads can end up in a livelock. 8. Why might livelock be considered a subtler design failure than deadlock, even though it's arguably less "catastrophic" in some ways?

**Production-Level:** 9. Describe a real-world networking or distributed-systems scenario where randomized backoff was specifically introduced to prevent a livelock-like problem. 10. How would you diagnose a suspected livelock in a production system where `jstack` shows no deadlock, but the system also isn't making progress?

---

## Follow-up Questions

- Does livelock require exactly two threads, like the classic examples? _(No — livelock can involve any number of threads whose mutual reactions prevent collective progress; two is simply the easiest case to illustrate.)_
- Can a livelock resolve itself over time, unlike a true deadlock? _(In practice, often yes — real systems have enough natural timing jitter (OS scheduling variance, etc.) that a perfectly synchronized livelock pattern often eventually breaks on its own; but this is NOT a guarantee, and relying on "it'll probably resolve itself eventually" is not an acceptable design strategy.)_
- Is a livelock a type of race condition? _(Not quite — it's a distinct category of concurrency problem; a race condition is about incorrect results from bad interleaving, while livelock is about complete lack of progress despite constant activity — though both stem from the same root category of "timing-dependent" concurrency issues.)_

---

## Best Practices

- Always use **randomized** delays in retry/backoff logic, never a fixed, deterministic delay, when multiple threads might be contending for the same resources.
- Consider exponential backoff (increasing delay ranges with each retry) combined with randomization for particularly contention-prone scenarios.
- When diagnosing a "stuck" production system, check for **both** deadlock (via `jstack`'s explicit detection) **and** livelock (via observing whether threads are `RUNNABLE` but making no measurable progress across repeated samples).
- Be cautious with "polite," cooperative retry logic — ironically, the more considerate the logic tries to be, the more careful you need to be about avoiding perfectly mirrored, unproductive interactions.

---

## Performance Notes

- Livelock is often **worse** for system resources than deadlock in one specific sense: livelocked threads consume real CPU cycles continuously while making zero progress, whereas deadlocked threads consume none — a livelock can look like a "CPU usage spike with no throughput," which is a particularly confusing production symptom.
- Randomized backoff fixes typically have negligible performance cost in the success case (the delay only applies on retry, and successful attempts usually happen quickly once desynchronized).

---

## Comparison Table

|Aspect|Deadlock|Livelock|
|---|---|---|
|Thread state|`BLOCKED`/`WAITING`|`RUNNABLE`|
|CPU usage|None from stuck threads|High — constant activity|
|Detected by `jstack`'s built-in deadlock check?|Yes|No|
|Typical cause|Inconsistent lock ordering|Non-randomized "polite" retry/backoff logic|
|Typical fix|Consistent lock ordering|Randomized backoff|

---

## Summary

- A livelock occurs when threads are actively executing (not blocked) but make no actual collective progress, typically due to each repeatedly reacting to the other's state in a mirrored, unproductive pattern.
- Unlike deadlock, livelocked threads remain `RUNNABLE` and consume CPU — making them invisible to standard deadlock-detection tools like `jstack`'s automatic cycle detection.
- The classic cause is "overly polite" retry logic that backs off deterministically; the standard fix is introducing **randomized** delays to break the lockstep pattern.
- Diagnosing livelock requires observing repeated, unproductive activity across multiple samples over time, rather than relying on a single snapshot or automatic tool detection.

---

## Cheat Sheet

|Symptom|Likely Issue|
|---|---|
|Threads `BLOCKED`, `jstack` shows deadlock cycle|Deadlock|
|Threads `RUNNABLE`, high CPU, but zero measured progress over time|Livelock|
|Fix: consistent lock ordering|Deadlock prevention|
|Fix: randomized backoff before retry|Livelock prevention|

---

## Keywords

Livelock, Randomized Backoff, Lockstep, Retry Logic, Exponential Backoff, RUNNABLE state

---

## Related Notes

[[36 - Deadlock]] [[38 - Starvation]] [[31 - CAS]] [[10 - Thread States]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's "no backoff" livelock-prone demo and observe (roughly) how many attempts each thread needs.
2. Reproduce the "randomized backoff" fixed version and confirm it typically resolves in far fewer attempts.

**Medium:** 3. Experiment with DETERMINISTIC (non-random) but non-zero backoff (e.g., always exactly 10ms) and observe whether this still occasionally produces livelock-like behavior compared to randomized backoff. 4. Modify the demo to track and print the total wall-clock time taken by the no-backoff version versus the randomized version.

**Hard:** 5. Design and implement a 3-thread livelock scenario (each politely backing off for the "next" thread in a cycle) and demonstrate that randomized backoff resolves it.

---

## Mini Project

**Project: Livelock vs. Deadlock Symptom Comparator**

Build a small console application that:

1. Implements both a deadlock scenario (from [[36 - Deadlock]]'s bank transfer example) and a livelock scenario (this note's "overly polite" resource contention example).
2. For each scenario, samples and logs thread states (`RUNNABLE`, `BLOCKED`, etc.) and rough CPU activity every second for 5 seconds.
3. Prints a clear side-by-side comparison showing the different "symptom signatures": the deadlock scenario shows `BLOCKED` threads with no CPU activity, while the livelock scenario shows `RUNNABLE` threads with continuous CPU activity but no progress.

**Goal:** Build direct, observable evidence of the key diagnostic distinction between these two failure modes — reinforcing the practical skill of telling them apart in a real debugging scenario.

---

## Common Interview Trap

**Trap Question:** _"If a production system appears completely stuck, and `jstack` shows no deadlock, can you conclude there's no concurrency bug involved?"_

The trap is assuming `jstack`'s automatic deadlock detection is a complete diagnostic — it isn't. The correct answer: **no**, you cannot conclude this. A **livelock** produces exactly this symptom: no detected deadlock cycle (since no thread is actually `BLOCKED` on a lock), yet the system still makes no real progress, because threads are stuck in an unproductive, mutually-reactive loop while remaining fully `RUNNABLE`. To properly rule out a concurrency issue in this scenario, you'd need to take **multiple** thread dumps over time and check whether the `RUNNABLE` threads are genuinely making progress (different stack traces/state each time) or are stuck repeating the same pattern (a livelock signature) — a single dump, or even `jstack`'s built-in deadlock check alone, is insufficient to rule this out.

---

## Frequently Asked Questions

**Q: Is livelock more or less common than deadlock in real production systems?** A: Deadlock is generally considered more common and more frequently discussed, partly because it's easier to detect and reason about; livelock is rarer in practice but notoriously harder to diagnose when it does occur, precisely because standard tooling doesn't automatically flag it.

**Q: Can a livelock eventually turn into (or reveal) a separate, genuine deadlock?** A: They're distinct phenomena, but a poorly-designed system could plausibly exhibit both types of issues in different code paths — they aren't mutually exclusive at the whole-system level, even though a single specific interaction is typically one or the other.

**Q: Does exponential backoff always fully prevent livelock?** A: It significantly reduces the _probability_ and _typical duration_ of livelock-like patterns by combining increasing delays with randomization, but no retry strategy can offer an absolute, mathematical guarantee against livelock in every conceivable pathological timing scenario — though in practice, well-randomized backoff is considered a highly effective, industry-standard mitigation.

**Q: Is livelock specific to Java, or a general distributed/concurrent systems concept?** A: It's a general concept, well-known in distributed systems and network protocol design (e.g., Ethernet collision backoff) — Java's specific manifestation (as shown in this note) is just one instance of a much broader, cross-domain problem category.

---

## Revision Checklist

- [ ] I can precisely define livelock and distinguish it from deadlock (thread state, CPU usage).
- [ ] I understand why `jstack`'s automatic deadlock detection cannot catch a livelock.
- [ ] I can explain the "overly polite" retry pattern that causes livelock.
- [ ] I know that randomized (not deterministic) backoff is the standard fix.
- [ ] I understand how to diagnose a suspected livelock using repeated thread dump sampling.

---

## Flashcards

**Q:** What is a livelock? **A:** A situation where threads are actively executing but make no actual collective progress, typically due to mutually reactive, unproductive behavior.

**Q:** What thread state do livelocked threads typically show? **A:** `RUNNABLE` — they are actively executing, unlike deadlocked threads which are `BLOCKED`.

**Q:** Does `jstack`'s automatic deadlock detection catch livelocks? **A:** No — since no thread is actually blocked waiting on a lock in a cycle.

**Q:** What is the standard fix for livelock-prone retry logic? **A:** Introduce randomized (not deterministic) delays before retrying.

**Q:** How would you diagnose a suspected livelock in production? **A:** Take multiple thread dumps over time and check if `RUNNABLE` threads are genuinely progressing or repeating the same unproductive pattern.

---

## References

- Oracle Java Tutorials — Liveness: https://docs.oracle.com/javase/tutorial/essential/concurrency/starvelive.html
- IEEE 802.3 Ethernet Standard — Collision Detection and Exponential Backoff (classic real-world livelock-avoidance precedent)
- `java.util.concurrent.ThreadLocalRandom` API Documentation