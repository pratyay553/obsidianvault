# notifyAll()

## Objective

By the end of this note, you will understand:

- Exactly what `Object.notifyAll()` does and how it differs from `notify()`
- Why it's the safer default choice for almost all intrinsic wait/notify scenarios
- The "thundering herd" performance consideration it introduces
- How to reason about when its slight inefficiency is an acceptable trade-off for correctness

This note completes the intrinsic wait/notify trilogy alongside [[40 - wait()]] and [[41 - notify()]].

---

## Why should I learn this?

`notifyAll()` is the method you'll use far more often than `notify()` in practice, precisely because of the danger `notify()` introduced in the previous note. Understanding exactly what `notifyAll()` guarantees — and its one real cost (waking threads that then immediately go back to sleep) — completes your understanding of Java's intrinsic signaling mechanism and equips you to make an informed choice, rather than a fear-driven "always use `notifyAll()` and never think about it again" default.

---

## Beginner Explanation

Recall the waiting-room analogy from [[41 - notify()]]: `notify()` calls out "next patient!" and picks **one, arbitrary** patient — with a real risk of calling the wrong one for what's actually available.

`notifyAll()` instead makes an announcement to the **entire waiting room**: "attention everyone, something has changed — please check if it's relevant to you." **Every** waiting patient gets up and checks. Most of them, if what changed wasn't relevant to their specific situation, simply sit back down (their `while` loop re-checks its condition, finds it still false, and calls `wait()` again). But the **one** patient for whom it actually is relevant also checks — and this time, correctly proceeds.

This is slightly less "efficient" (everyone had to get up and check, even those who didn't need to), but it's **guaranteed correct** — the relevant patient is never missed.

---

## Real World Analogy

**Fire Alarm Analogy**

Imagine a fire alarm in a large office building. When it sounds, **every** occupant, regardless of what they were doing, stops and checks the situation — most (correctly) determine it's a routine drill or false alarm and return to their desks; the handful who actually need to evacuate for a real reason do so. A "smart" alarm that only alerted **one, arbitrarily chosen** occupant (analogous to `notify()`) would be far riskier — what if the chosen occupant wasn't even near the affected area, while the people who genuinely needed to know remained completely unaware? The inefficiency of "everyone checks, most go back to what they were doing" is a small, acceptable cost for the safety guarantee that nobody who needed to know is ever missed.

---

## Technical Explanation

### Method Signature

```java
public final void notifyAll()
```

Same precondition as `wait()`/`notify()`: the calling thread must hold the object's monitor, or `IllegalMonitorStateException` is thrown.

### What `notifyAll()` Guarantees

- **Every** thread currently in the object's wait set is moved from "waiting" to "eligible to attempt re-acquiring the monitor."
- Each of these threads will, **one at a time** (since only one thread can hold the monitor at once), re-acquire the lock, re-check its `while` condition, and either proceed (if its condition is now true) or call `wait()` again (if not).
- **No thread that was genuinely waiting for the change that just occurred will be missed** — this is the core safety guarantee `notifyAll()` provides that `notify()` cannot.

### Why It's Slightly Less Efficient — The "Thundering Herd" Consideration

If 50 threads are waiting on the same monitor, and only 1 of them can actually proceed after a given `notifyAll()`, all 50 will wake up, and 49 of them will do a small amount of wasted work: re-acquire the lock (waiting their turn), re-check the condition, find it false, release the lock, and go back to `wait()`. This pattern — many threads waking for one relevant change — is sometimes called the **"thundering herd" problem**, and it's the specific, measurable cost of choosing `notifyAll()`'s safety over `notify()`'s efficiency.

For most applications, with modest numbers of waiting threads, this cost is negligible and entirely worth the correctness guarantee. It becomes a genuine performance concern primarily in scenarios with **very large numbers** of waiting threads competing for infrequently-changing conditions — precisely the scenario where separate `Condition` objects (see [[35 - Condition]]) become valuable, letting you achieve `notifyAll()`'s safety with `notify()`-like targeted efficiency, by separating genuinely different wait reasons into different, independent wait-sets.

### Full, Correct Producer-Consumer Recap (from [[39 - Producer Consumer]])

```java
public synchronized void put(T item) throws InterruptedException {
    while (queue.size() == capacity) {
        wait();
    }
    queue.add(item);
    notifyAll(); // safe -- wakes BOTH other producers (irrelevant, will re-wait) AND consumers (relevant!)
}

public synchronized T take() throws InterruptedException {
    while (queue.isEmpty()) {
        wait();
    }
    T item = queue.poll();
    notifyAll(); // safe -- wakes BOTH other consumers (irrelevant) AND producers (relevant!)
    return item;
}
```

This is exactly the safe, correct pattern from [[39 - Producer Consumer]] — using `notifyAll()` here guarantees that regardless of how many producers and consumers are mixed in the shared wait set, the "relevant" ones are never missed, at the small cost of occasionally also waking irrelevant ones.

---

## Internal Working

Step-by-step of `someObject.notifyAll()`:

1. Precondition check: calling thread holds the monitor, or `IllegalMonitorStateException`.
2. **Every** thread currently in `someObject`'s wait set is marked as eligible to compete for re-acquiring the monitor.
3. The calling thread continues executing (still holding the monitor) until it exits its `synchronized` block, at which point the monitor becomes available.
4. The now-eligible threads compete (via the OS scheduler, with no fairness guarantee by default — see [[38 - Starvation]]) to re-acquire the monitor, **one at a time**.
5. Each, upon re-acquiring, re-checks its `while` condition: if true, proceeds; if false, releases the monitor again and calls `wait()` once more, re-joining the wait set.

---

## Memory Diagram

```
        Object monitor
   -----------------------------------
   |  wait set: [P1, C1, P2, C2, C3]    |  (Producers and Consumers mixed)
   -----------------------------------

   Consumer calls take(), removes item, calls notifyAll():
   -----------------------------------
   |  wait set: []  (ALL moved to "eligible")            |
   -----------------------------------

   One by one, each re-acquires the lock:
   P1: re-checks "full?" -> NO now -> proceeds!
   C1: re-checks "empty?" -> still empty -> wait() again
   P2: re-checks "full?" -> NO (P1 already added) -> proceeds? depends on capacity
   C2, C3: re-check -> likely still empty -> wait() again
```

---

## Execution Flow

```
Thread calls notifyAll()
        |
        v
ALL waiting threads become eligible
        |
        v
Threads compete, one at a time, to re-acquire the monitor
        |
        v
Each re-checks its OWN while condition upon re-acquiring
        |
   -----------------------------
   |                           |
 condition true             condition false
   |                           |
   v                           v
Proceeds normally          Calls wait() again,
                            rejoins the wait set
```

---

## Code Example

```java
public class NotifyAllDemo {

    static final Object lock = new Object();
    static int resourceCount = 0;

    public static void main(String[] args) throws InterruptedException {

        Runnable waitsForEven = () -> {
            synchronized (lock) {
                while (resourceCount % 2 != 0) {
                    try {
                        System.out.println(Thread.currentThread().getName() + " waiting for EVEN...");
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                System.out.println(Thread.currentThread().getName() + " proceeded! count=" + resourceCount);
            }
        };

        Runnable waitsForOdd = () -> {
            synchronized (lock) {
                while (resourceCount % 2 == 0) {
                    try {
                        System.out.println(Thread.currentThread().getName() + " waiting for ODD...");
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                System.out.println(Thread.currentThread().getName() + " proceeded! count=" + resourceCount);
            }
        };

        Thread evenWaiter = new Thread(waitsForEven, "EvenWaiter");
        Thread oddWaiter = new Thread(waitsForOdd, "OddWaiter");

        evenWaiter.start();
        Thread.sleep(50);
        oddWaiter.start();
        Thread.sleep(50);

        synchronized (lock) {
            resourceCount = 1; // ODD
            System.out.println("Main: set count=1, calling notifyAll() (SAFE -- wakes BOTH to re-check)");
            lock.notifyAll(); // BOTH wake, BOTH re-check; only OddWaiter's condition is true
        }

        evenWaiter.join(2000);
        oddWaiter.join(2000);

        System.out.println("EvenWaiter still alive (still waiting, correctly)? " + evenWaiter.isAlive());
        System.out.println("OddWaiter still alive (should be FALSE -- it should have proceeded)? " + oddWaiter.isAlive());
    }
}
```

### Line-by-line explanation

1. This is the **exact same** heterogeneous-waiter scenario from [[41 - notify()]]'s demo, but using `notifyAll()` instead.
2. Both `EvenWaiter` and `OddWaiter` wake up when `notifyAll()` is called — `EvenWaiter` re-checks, finds `resourceCount % 2 != 0` still true (it's odd), and correctly goes back to waiting; `OddWaiter` re-checks, finds its condition satisfied, and proceeds.
3. **Unlike** the `notify()` version, this outcome is **reliable, every single run** — `OddWaiter` will always correctly proceed, and `EvenWaiter` will always correctly remain waiting (since the count is genuinely odd and never becomes even in this program) — no "unlucky" arbitrary-selection risk exists here.

---

## Dry Run

```
EvenWaiter waiting for EVEN...
OddWaiter waiting for ODD...
Main: set count=1, calling notifyAll() (SAFE -- wakes BOTH to re-check)
OddWaiter proceeded! count=1
EvenWaiter still alive (still waiting, correctly)? true
OddWaiter still alive (should be FALSE -- it should have proceeded)? false
```

This exact output is **reliable and repeatable** — a direct contrast to [[41 - notify()]]'s non-deterministic demo.

---

## Common Mistakes

1. **Assuming `notifyAll()` guarantees all threads run in parallel or in any particular order.**
    
    - They still must take turns re-acquiring the single monitor, one at a time — `notifyAll()` only guarantees all get a **chance** to re-check, not simultaneous execution or any specific ordering among them.
2. **Overusing `notifyAll()` in extremely high-waiter-count, high-frequency-change scenarios without considering the thundering herd cost.**
    
    - In such specific, performance-critical scenarios, consider separate `Condition` objects (see [[35 - Condition]]) to combine safety with better targeting.
3. **Forgetting the `while` loop is still mandatory, even with `notifyAll()`.**
    
    - `notifyAll()` solves the "wrong thread woken" problem, but does **not** eliminate the need to re-verify the condition — spurious wakeups and multi-waiter races (see [[40 - wait()]]) still require the `while` loop regardless of which notify method is used.
4. **Reflexively avoiding `notifyAll()` "for performance," without measuring whether it's actually a bottleneck.**
    
    - Premature optimization here risks reintroducing `notify()`'s correctness danger for a performance gain that may not even be measurable in your actual system's typical waiter counts.

---

## Production Usage

- **The default, recommended choice** for the vast majority of hand-rolled intrinsic wait/notify code, specifically because of the correctness guarantee over `notify()`'s narrow, easily-violated safety precondition.
- **Bounded buffers, connection pools, and other shared-resource coordination** implemented with intrinsic locking almost universally use `notifyAll()` unless a rigorous, documented justification for `notify()` exists.
- **Frameworks and libraries** that need finer-grained, more efficient signaling typically move to `ReentrantLock` + separate `Condition` objects (see [[35 - Condition]]) rather than trying to optimize `notify()`/`notifyAll()` usage further within the intrinsic locking model.

---

## Interview Questions

**Beginner:**

1. What does `Object.notifyAll()` do?
2. How does `notifyAll()` differ from `notify()`?
3. Do all threads woken by `notifyAll()` run simultaneously?

**Intermediate:** 4. What is the "thundering herd" problem, and how does it relate to `notifyAll()`? 5. Is the `while` loop still necessary when using `notifyAll()`? Why? 6. Why is `notifyAll()` generally considered the safer default over `notify()`?

**Advanced:** 7. In what specific scenario might `notifyAll()`'s thundering herd cost become a genuine, measurable performance concern? 8. How do separate `Condition` objects let you achieve `notifyAll()`-like safety with better, `notify()`-like targeting efficiency?

**Production-Level:** 9. Describe a code review policy you'd establish regarding the choice between `notify()` and `notifyAll()`. 10. At what point (in terms of waiter count or system scale) would you consider migrating from `notifyAll()`-based intrinsic locking to `Condition`-based signaling for performance reasons?

---

## Follow-up Questions

- Does `notifyAll()` clear the entire wait set immediately, all at once? _(It marks all waiting threads as eligible, but they still individually, sequentially compete to re-acquire the single monitor — there's no literal "all at once" execution, since only one thread can hold the monitor at a time.)_
- If `notifyAll()` is called with an empty wait set, what happens? _(Nothing — exactly like `notify()`, it's simply a no-op; there's no threads to wake, and no notification is stored for the future.)_
- Can `notifyAll()` cause a performance problem even with `Condition`-based signaling? _(Yes, in principle — `Condition.signalAll()` has the analogous thundering-herd characteristic for its own specific wait-set; the benefit of `Condition` is having _smaller, more targeted_ wait-sets in the first place, reducing how many threads are unnecessarily woken by any single `signalAll()`, not eliminating the underlying trade-off entirely.)_

---

## Best Practices

- Default to `notifyAll()` for any intrinsic wait/notify code unless you've rigorously verified the narrow homogeneous-waiter precondition required for `notify()`'s safety.
- Always retain the `while` loop guarding the wait condition, regardless of which notify method is used.
- Consider `Condition` objects (see [[35 - Condition]]) if profiling reveals `notifyAll()`'s thundering herd cost is a genuine bottleneck in a high-waiter-count scenario.
- Document any use of `notify()` (instead of `notifyAll()`) with a clear justification, since it represents a deliberate, easily-violated safety trade-off that future maintainers need to understand.

---

## Performance Notes

- `notifyAll()`'s cost scales with the number of currently-waiting threads — in typical applications with modest waiter counts (a handful to a few dozen), this cost is negligible.
- In systems with hundreds or thousands of waiting threads on a single monitor, the thundering herd effect can become measurable — this is precisely the scale at which `Condition`-based, more finely-partitioned signaling starts to show a clear advantage.

---

## Comparison Table

|Aspect|`notify()`|`notifyAll()`|
|---|---|---|
|Threads woken|One (arbitrary)|All|
|Safe with heterogeneous waiters|No|Yes|
|"Thundering herd" cost|None|Yes (proportional to waiter count)|
|Recommended default|Only with rigorous justification|Yes, for most cases|

---

## Summary

- `notifyAll()` wakes **every** thread currently waiting on the object's monitor, letting each independently re-check its own condition.
- This guarantees no genuinely-relevant waiting thread is ever missed, unlike `notify()`'s arbitrary single-thread selection.
- The cost is the "thundering herd" effect: irrelevant threads also wake, re-check, and go back to waiting — a real but usually negligible overhead.
- The `while` loop guarding the wait condition remains mandatory regardless of which notify method is used.
- `notifyAll()` is the safe, recommended default; reach for `notify()` only with rigorous, documented justification, and consider `Condition` objects for better efficiency at scale without sacrificing safety.

---

## Cheat Sheet

|Question|Answer|
|---|---|
|How many threads does `notifyAll()` wake?|All currently waiting|
|Do they run in parallel?|No — they still take turns re-acquiring the monitor|
|Is the `while` loop still needed?|Yes, always|
|Main cost|"Thundering herd" — irrelevant threads also wake and re-check|
|Recommended default?|Yes, over `notify()`, unless rigorously justified otherwise|

---

## Keywords

notifyAll, Thundering Herd, Wait Set, Heterogeneous Waiters, Safe Default, Condition

---

## Related Notes

[[40 - wait()]] [[41 - notify()]] [[35 - Condition]] [[39 - Producer Consumer]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `NotifyAllDemo` and confirm the reliable, correct outcome across multiple runs.
2. Compare this directly against re-running [[41 - notify()]]'s demo several times, noting the difference in reliability.

**Medium:** 3. Extend the demo to 3 different waiter "kinds" (e.g., waiting for count%3==0, ==1, ==2) and confirm `notifyAll()` correctly handles all three reliably. 4. Add a counter tracking how many total "wasted" wakeups (threads that woke, re-checked, and went back to waiting) occur across a longer-running simulation, to quantify the thundering herd cost.

**Hard:** 5. Build a benchmark comparing a `notifyAll()`-based bounded buffer against a `Condition`-based one (from [[35 - Condition]]) under a scenario with many producer and consumer threads, measuring the difference in wasted wakeup overhead.

---

## Mini Project

**Project: Thundering Herd Cost Visualizer**

Build a small console application that:

1. Creates a configurable number of "waiter" threads (e.g., 5, 20, 100), all waiting on the same monitor for a shared counter to reach a specific target value.
2. Repeatedly updates the counter and calls `notifyAll()`, logging how many threads wake and re-check versus how many actually proceed on each notification.
3. Runs this across different waiter counts and reports the ratio of "wasted" wakeups to "productive" wakeups for each configuration.
4. Includes a brief written conclusion on when this overhead becomes significant enough to justify moving to `Condition`-based signaling.

**Goal:** Build a concrete, measured sense of the thundering herd trade-off, moving beyond "notifyAll is slightly less efficient" into actual quantified understanding.

---

## Common Interview Trap

**Trap Question:** _"If `notifyAll()` wakes every waiting thread, doesn't that mean my program is doing unnecessary work every single time, and I should avoid it whenever possible for performance?"_

The trap is treating `notifyAll()`'s "wasted wakeup" cost as a universal, always-significant performance problem that should drive design decisions by default. The correct, more nuanced answer: for the **vast majority** of real applications — where the number of concurrently waiting threads on any given monitor is small (a handful to a few dozen) — this overhead is genuinely negligible and not worth the correctness risk of trying to use `notify()` instead. The thundering herd cost only becomes a meaningful, measurable concern at **significant scale** (hundreds or more waiting threads on a single monitor), and even then, the correct response is typically to **redesign** the signaling (e.g., using separate `Condition` objects to reduce each individual wait-set's size), not to gamble on `notify()`'s fragile, easily-violated safety precondition. "Avoid `notifyAll()` for performance" without first establishing genuine, measured need is a classic case of premature optimization trading away correctness for an unproven benefit.

---

## Frequently Asked Questions

**Q: Does `notifyAll()` release the monitor immediately when called?** A: No — the calling thread continues to hold the monitor until it exits its `synchronized` block (or otherwise releases it); `notifyAll()` only marks the waiting threads as eligible to compete for the monitor once it does become available.

**Q: Is there a `notifyAll()` equivalent for `ReentrantLock`/`Condition`?** A: Yes — `Condition.signalAll()` is the direct equivalent, waking all threads waiting on that specific `Condition` object (see [[35 - Condition]]).

**Q: Does calling `notifyAll()` multiple times in a row have any cumulative or different effect compared to calling it once?** A: If the wait set is already empty after the first call (all threads have moved on), subsequent calls before any new `wait()` calls are simply no-ops; there's no "extra" effect from calling it redundantly.

**Q: Can `notifyAll()` cause a `ConcurrentModificationException` or similar issue if new threads call `wait()` while it's in progress?** A: No — the JVM's internal monitor implementation safely handles this; `notifyAll()` only affects threads that were in the wait set at the moment it was called, and any new `wait()` calls happening concurrently are handled safely by the JVM's internal synchronization, not something application code needs to guard against directly.

---

## Revision Checklist

- [ ] I understand that `notifyAll()` wakes every currently-waiting thread, not just one.
- [ ] I understand why this is the safer default compared to `notify()`.
- [ ] I understand the "thundering herd" cost and roughly when it becomes significant.
- [ ] I know the `while` loop remains mandatory regardless of which notify method is used.
- [ ] I can explain how `Condition` objects offer a way to get `notifyAll()`'s safety with better targeting at scale.

---

## Flashcards

**Q:** How many threads does `notifyAll()` wake? **A:** All threads currently waiting on that object's monitor.

**Q:** Do threads woken by `notifyAll()` run in parallel? **A:** No — they still take turns re-acquiring the single monitor, one at a time.

**Q:** What is the "thundering herd" problem? **A:** Many threads waking up in response to a `notifyAll()`, when only one (or a few) actually have a satisfied condition — most simply re-check and go back to waiting.

**Q:** Is `notifyAll()` generally considered the safer default over `notify()`? **A:** Yes.

**Q:** Is the `while` loop still required with `notifyAll()`? **A:** Yes — always, regardless of which notify method is used.

---

## References

- Oracle Java Tutorials — Guarded Blocks: https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html
- `java.lang.Object` API Documentation (`notify`, `notifyAll`, `wait`)
- JLS (Java Language Specification) — Chapter 17: Threads and Locks