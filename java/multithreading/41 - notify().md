# notify()

## Objective

By the end of this note, you will understand:

- Exactly what `Object.notify()` does, and its precise, limited guarantee
- Why you cannot control (or even know) which specific waiting thread gets woken
- The specific, narrow scenarios where `notify()` is safe to use instead of `notifyAll()`
- Why defaulting to `notifyAll()` is the safer choice in almost all real code

This note builds directly on [[40 - wait()]] and pairs with [[42 - notifyAll()]] to complete the intrinsic signaling trio.

---

## Why should I learn this?

`notify()` looks like the natural, symmetric counterpart to `wait()` — but it has a sharp, easy-to-misuse limitation: it wakes exactly **one, arbitrarily-chosen** thread from the object's wait set, with **no** control over which one. Using it incorrectly (when multiple different "kinds" of waiting threads share the same wait set, as in [[39 - Producer Consumer]]) can cause a genuine, hard-to-diagnose stall — not just a minor inefficiency. Understanding precisely when `notify()` is actually safe is a mark of real concurrency understanding.

---

## Beginner Explanation

Imagine a doctor's waiting room where several patients are waiting for **different** reasons — some for an X-ray, some for lab results, some for a prescription pickup — but they're all sitting in the **same** waiting room with just **one shared "next!" call**. If the receptionist calls "next!" without saying who or what for, whichever patient happens to react and stand up first might discover the call had nothing to do with them, and they awkwardly sit back down — meanwhile, the patient who **actually** needed to be called might still be sitting there, unaware anything happened at all.

`notify()` is exactly this "single, undirected 'next!' call" — it wakes up **one** waiting thread, chosen arbitrarily by the JVM, with no way to target a specific "kind" of waiter. If that's not the thread that can actually make progress, the system can stall.

---

## Real World Analogy

**Warehouse Analogy**

Imagine a warehouse intercom system with just one shared channel used by workers waiting for entirely different things: some waiting for a forklift to become available, others waiting for a specific truck to arrive at the loading dock. If a manager announces "attention, something is now available!" over this one shared channel (a `notify()`), only **one** worker (chosen arbitrarily) reacts and checks — and if that worker happens to be someone waiting for the truck (which hasn't actually arrived), they'll just go back to waiting, while the forklift-waiting worker who could have actually used the now-available forklift never got the memo.

A well-designed warehouse would use **separate** channels for separate purposes (this is exactly what `Condition` objects — see [[35 - Condition]] — provide) rather than relying on one shared, imprecise announcement system.

---

## Technical Explanation

### Method Signature

```java
public final native void notify();
```

Like `wait()`, `notify()` is defined on `Object` and must be called while holding the target object's monitor — otherwise it throws `IllegalMonitorStateException`.

### What `notify()` Guarantees, Precisely

1. If the object's wait set is empty (no threads currently waiting), `notify()` does **nothing** — the "notification" is simply lost, with no lasting effect.
2. If the wait set has one or more threads, `notify()` wakes up **exactly one** of them, chosen by the JVM using an **unspecified** selection mechanism — you cannot predict, influence, or rely on which specific thread will be chosen.
3. The woken thread must still **re-acquire** the monitor (see [[40 - wait()]]) before its `wait()` call actually returns — `notify()` itself does not release the caller's hold on the lock; that only happens when the notifying thread's own `synchronized` block ends.

### The Danger: When Multiple "Kinds" of Waiters Share One Wait Set

Recall the Producer-Consumer example from [[39 - Producer Consumer]], where **both** producers (waiting for "not full") and consumers (waiting for "not empty") call `wait()` on the **same** shared object. If a producer, after adding an item, called `notify()` instead of `notifyAll()`:

- The JVM might wake **another waiting producer** (also waiting for "not full") instead of a waiting consumer.
- That woken producer re-checks its own condition ("is it still full?") — if the buffer is genuinely no longer full, it might proceed. But if it happens to check "is it empty?" logic incorrectly, or if there were actually consumers waiting that never got woken, the **intended** effect (waking a consumer to process the newly available item) might never happen.
- In the worst case, with enough unlucky arbitrary selections, this can leave threads waiting **indefinitely**, even though progress was theoretically possible — a genuine correctness bug, not just an inefficiency.

### When `notify()` IS Safe to Use

`notify()` is safe **only** when **both** of these conditions hold:

1. **All threads in the wait set are waiting for the exact same condition** (i.e., any one of them being woken is equally "correct").
2. **Waking exactly one thread is sufficient** — i.e., the state change that triggered the notification only creates an opportunity for one waiter to proceed, not multiple.

```java
// SAFE example: a single-slot "mailbox" where all waiters want the SAME thing
class SingleSlotMailbox {
    private String message = null;

    public synchronized void post(String msg) {
        message = msg;
        notify(); // safe: all waiters want the exact same "message arrived" event,
                  // and only one waiter can consume this one message anyway
    }

    public synchronized String take() throws InterruptedException {
        while (message == null) {
            wait();
        }
        String result = message;
        message = null;
        return result;
    }
}
```

Even here, some teams still default to `notifyAll()` for extra safety/robustness against future code changes (e.g., someone later adding a different kind of waiter) — this is a common, reasonable defensive practice.

---

## Internal Working

Step-by-step of `notify()`:

1. The calling thread must hold the target object's monitor.
2. The JVM selects **one** thread from the object's internal wait set, using an implementation-defined (unspecified, non-guaranteed) mechanism.
3. That thread is removed from the wait set and marked as eligible to compete for re-acquiring the monitor.
4. The notifying thread continues executing (it does **not** immediately give up the lock — only when its own `synchronized` block ends).
5. Once the notifying thread releases the lock, the woken thread (along with any other threads also trying to acquire it) competes to acquire it; whichever succeeds resumes its `wait()` call, re-checking its `while` condition.

---

## Memory Diagram

```
        Object's wait set (BEFORE notify)
   -----------------------------------
   |  [Producer-Waiting-1,             |
   |   Consumer-Waiting-1,             |
   |   Producer-Waiting-2]             |
   -----------------------------------

   notify() called -- JVM picks ONE, arbitrarily:
   -----------------------------------
   |  Selected: Producer-Waiting-2      |  <- could have been ANY of the three!
   -----------------------------------
   |  Remaining wait set:               |
   |  [Producer-Waiting-1,              |
   |   Consumer-Waiting-1]              |
   -----------------------------------

   If the buffer was actually non-empty (needing a CONSUMER),
   but a PRODUCER was arbitrarily woken instead --
   the intended progress may not happen.
```

---

## Execution Flow

```
Thread calls notify() (holding the monitor)
        |
        v
JVM selects ONE arbitrary thread from the wait set
        |
        v
That thread is marked eligible (still needs to re-acquire monitor)
        |
        v
Notifying thread continues, eventually releases monitor
        (exiting its synchronized block)
        |
        v
Selected thread (and any other lock-seeking threads)
compete to acquire the monitor
        |
        v
Winner resumes its wait() call, re-checks while condition
```

---

## Code Example

```java
public class NotifyDemo {

    static final Object lock = new Object();
    static String mailboxMessage = null;

    public static void main(String[] args) throws InterruptedException {

        Runnable waiterTask = () -> {
            synchronized (lock) {
                while (mailboxMessage == null) {
                    try {
                        System.out.println(Thread.currentThread().getName() + " waiting for message...");
                        lock.wait();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                System.out.println(Thread.currentThread().getName() + " received: " + mailboxMessage);
            }
        };

        Thread waiterA = new Thread(waiterTask, "Waiter-A");
        Thread waiterB = new Thread(waiterTask, "Waiter-B");

        waiterA.start();
        waiterB.start();
        Thread.sleep(200); // ensure both are waiting

        synchronized (lock) {
            mailboxMessage = "Hello!";
            System.out.println("Main: posting message, calling notify() (only ONE waiter will wake)...");
            lock.notify(); // only ONE of Waiter-A / Waiter-B wakes -- unpredictable which
        }

        waiterA.join(2000);
        waiterB.join(2000);

        if (waiterA.isAlive() || waiterB.isAlive()) {
            System.out.println("NOTE: one waiter is STILL WAITING -- notify() only wakes one!");
        }
    }
}
```

### Line-by-line explanation

1. Both `Waiter-A` and `Waiter-B` wait on the exact same condition (`mailboxMessage == null`) — this is actually a "safe" scenario for `notify()`, since both waiters want the same thing.
2. `lock.notify()` wakes **exactly one** of them — you cannot predict which from run to run.
3. The **other** waiter remains waiting indefinitely in this particular demo (since `mailboxMessage` is only ever set once, and only one `notify()` is called) — demonstrating concretely that `notify()` does **not** wake everyone, only one.
4. This is intentionally left "unfinished" to make the point vivid: if you needed **both** waiters to eventually proceed, `notify()` alone would leave one of them stuck forever in this exact setup — you'd need `notifyAll()` instead.

---

## Dry Run

**Possible Output 1:**

```
Waiter-A waiting for message...
Waiter-B waiting for message...
Main: posting message, calling notify() (only ONE waiter will wake)...
Waiter-A received: Hello!
NOTE: one waiter is STILL WAITING -- notify() only wakes one!
```

**Possible Output 2 (different arbitrary choice):**

```
Waiter-A waiting for message...
Waiter-B waiting for message...
Main: posting message, calling notify() (only ONE waiter will wake)...
Waiter-B received: Hello!
NOTE: one waiter is STILL WAITING -- notify() only wakes one!
```

Both are valid — the JVM's choice of which thread to wake is unspecified and can vary from run to run.

---

## Common Mistakes

1. **Using `notify()` when multiple threads are waiting for DIFFERENT conditions on the same object.**
    
    - As explained, this can leave the "correct" waiter perpetually un-woken, causing a genuine, hard-to-diagnose stall — this is the single most dangerous misuse.
2. **Assuming `notify()` wakes the "longest-waiting" or "most relevant" thread.**
    
    - There is **no** such guarantee — the selection is entirely unspecified by the JVM specification; don't build any logic around assumptions of fairness or relevance.
3. **Using `notify()` "for efficiency" without carefully verifying both safety conditions (same wait condition, single-waiter-suffices) actually hold.**
    
    - Premature optimization here is a common source of subtle, intermittent production bugs — always default to `notifyAll()` unless you've carefully verified `notify()`'s safety conditions.
4. **Not realizing that a "lost" notification (calling `notify()` when no one is waiting) has no lasting effect.**
    
    - If a thread calls `wait()` **after** a `notify()` was already called (missing the window), it will simply wait — potentially forever, if no further `notify()`/`notifyAll()` ever comes — this is why the underlying **state** (the condition itself, like `mailboxMessage != null`), not just the notification event, must always be what the `while` loop checks.

---

## Production Usage

- **`notify()` alone is rarely used in real production code** — the risk of subtly wrong behavior when requirements evolve (someone adds a new kind of waiter later) makes `notifyAll()` the far more common, defensive default.
- **Single-purpose, single-slot coordination primitives** (like the mailbox example) are among the few legitimate places `notify()` might reasonably appear, though even here, many teams prefer `notifyAll()` for long-term safety.
- **`Condition`-based code (see [[35 - Condition]])** effectively replaces most legitimate `notify()` use cases with a **safer** mechanism: separate `Condition` objects mean `signal()` (the `Condition` equivalent of `notify()`) only ever wakes threads waiting for that **specific** condition, eliminating the "wrong kind of waiter woken" risk entirely.

---

## Interview Questions

**Beginner:**

1. What does `Object.notify()` do?
2. Can you control or predict which specific waiting thread gets woken by `notify()`?
3. What happens if `notify()` is called when no thread is currently waiting?

**Intermediate:** 4. Under what two conditions is it safe to use `notify()` instead of `notifyAll()`? 5. What could go wrong if `notify()` is used when multiple different kinds of waiters share the same wait set? 6. Does `notify()` immediately release the calling thread's hold on the monitor?

**Advanced:** 7. Explain, with the Producer-Consumer scenario, exactly how misusing `notify()` could cause an indefinite stall. 8. How does `Condition`'s `signal()` method provide a safer alternative to `notify()` in scenarios with multiple distinct waiter types?

**Production-Level:** 9. Why is `notifyAll()` generally preferred as the safer default in production code, even when `notify()` might theoretically be "more efficient"? 10. Describe a real scenario where a codebase using `notify()` safely at first later broke due to a new kind of waiter being added, and how you'd prevent this class of regression.

---

## Follow-up Questions

- Does `notify()` guarantee the woken thread will run immediately? _(No — it only makes the thread eligible; it must still compete to re-acquire the monitor, and actual scheduling depends on the OS scheduler, see [[11 - Thread Scheduler]].)_
- If two threads are waiting and `notify()` is called twice in a row, are both eventually woken? _(Yes — each `notify()` call wakes one thread from the wait set, assuming there are still threads left waiting; calling it twice would wake up to two different threads, one per call, assuming both are still present in the wait set.)_
- Is there any way to specify WHICH thread `notify()` should wake? _(No — this is exactly `notify()`'s core limitation; if you need targeted waking, use separate `Condition` objects instead, see [[35 - Condition]].)_

---

## Best Practices

- Default to `notifyAll()` unless you have carefully and explicitly verified both of `notify()`'s safety conditions hold for your specific scenario.
- If you're tempted to use `notify()` for a perceived efficiency gain, consider whether `Condition`-based signaling (see [[35 - Condition]]) would provide the same efficiency more safely, via targeted `signal()` calls on separate conditions.
- Document clearly, in code comments, WHY `notify()` is safe in any place you do use it — future maintainers need to understand the specific invariant that makes it safe, so they don't accidentally break it.

---

## Performance Notes

- `notify()` is marginally cheaper than `notifyAll()` when there are many waiting threads, since it only needs to wake one rather than all of them.
- This efficiency gain is only worth pursuing when its safety conditions are genuinely, carefully verified — the risk of a subtle correctness bug from misuse far outweighs the modest performance benefit in the vast majority of real-world cases.

---

## Comparison Table

|Aspect|`notify()`|`notifyAll()` (see [[42 - notifyAll()]])|
|---|---|---|
|Threads woken|Exactly one (arbitrary)|All waiting threads|
|Safe with mixed waiter types?|No — risky|Yes — always safe|
|Efficiency|Slightly better (fewer wakeups)|Slightly worse (all wake, most re-check and re-sleep)|
|Recommended default|No|Yes|

---

## Summary

- `Object.notify()` wakes exactly **one**, arbitrarily-selected thread from the object's wait set — with no way to control or predict which one.
- It's only safe to use when all potential waiters share the exact same wait condition, AND waking just one is sufficient to make correct progress.
- Misusing `notify()` when multiple different "kinds" of waiters share a wait set can cause a genuine, hard-to-diagnose indefinite stall — not just an efficiency loss.
- `notifyAll()` is the safer default in almost all real code; `Condition`'s `signal()` (paired with separate `Condition` objects) provides a genuinely safer, targeted alternative when precision matters.

---

## Cheat Sheet

|Rule|Detail|
|---|---|
|Threads woken|Exactly one, chosen arbitrarily by the JVM|
|Safe when|All waiters share the same condition, AND one waking suffices|
|Unsafe when|Multiple different waiter "kinds" share one wait set|
|Default choice|`notifyAll()`, unless safety is explicitly verified|
|No one waiting?|Notification has no lasting effect — it's simply lost|

---

## Keywords

notify, Arbitrary Selection, Wait Set, IllegalMonitorStateException, Single-Slot Mailbox, notifyAll

---

## Related Notes

[[40 - wait()]] [[42 - notifyAll()]] [[39 - Producer Consumer]] [[35 - Condition]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `NotifyDemo` and confirm one waiter remains stuck while the other receives the message.
2. Change `notify()` to `notifyAll()` and confirm both waiters now correctly wake and (since only one can actually consume the single message) one proceeds while the other correctly loops back to waiting on an updated, still-valid condition (or exits gracefully, depending on your loop design).

**Medium:** 3. Design and implement a "safe" `notify()` use case (all waiters share an identical condition, single wakeup suffices) and justify in comments why it's safe. 4. Deliberately misuse `notify()` in a Producer-Consumer scenario (as warned about in this note) and observe the resulting stall with multiple producers and consumers.

**Hard:** 5. Build a small test harness that runs a `notify()`-based coordination scenario many times, tracking how often (if ever) it produces an incorrect/stalled outcome versus a `notifyAll()`-based version of the same scenario, run equally often.

---

## Mini Project

**Project: notify() Safety Auditor**

Build a small console application that:

1. Implements two versions of a coordination primitive: one using `notify()`, one using `notifyAll()`, both intended for a scenario with 2 different "kinds" of waiters (e.g., "waiting for A" and "waiting for B").
2. Runs each version repeatedly (e.g., 50 times) under randomized timing, tracking whether all expected waiters eventually get woken and complete.
3. Reports the failure rate for the `notify()` version (which should show at least some stalls) versus the `notifyAll()` version (which should show zero).

**Goal:** Build direct, statistical, hands-on proof of exactly why `notify()` is dangerous with mixed waiter types — turning an abstract warning into measured, undeniable evidence.

---

## Common Interview Trap

**Trap Question:** _"If I'm certain only one thread will ever be waiting at a time in my system, is it always safe to use `notify()` instead of `notifyAll()`?"_

The trap is treating "only one waiter right now" as a permanent, unchangeable invariant. The correct, more cautious answer: **it might be safe today, but it's fragile** — if the codebase evolves (someone adds a second producer, a monitoring thread that also waits on the same object, etc.), the "only one waiter" assumption can silently break, and `notify()`'s behavior can become incorrect without any obvious signal that something changed. Since verifying "will this always, forever, only ever have one waiter, across all future code changes" is a much harder guarantee to maintain than it initially seems, many experienced engineers treat `notifyAll()` as the safe, low-maintenance-burden default, reserving `notify()` only for cases with extremely well-documented, actively-guarded invariants — and even then, often still preferring `notifyAll()` for the peace of mind.

---

## Frequently Asked Questions

**Q: Does `notify()` guarantee FIFO (first-in-first-out) ordering among waiting threads?** A: No — the JVM specification does not mandate any particular ordering; you cannot rely on "the thread that's been waiting longest" being the one selected.

**Q: Is there a way to notify a SPECIFIC thread, rather than an arbitrary one from the wait set?** A: Not directly via intrinsic `notify()` — this is precisely the gap that separate `Condition` objects (see [[35 - Condition]]) address, by letting you maintain multiple, independent wait sets so you can target which "group" of waiters gets signaled.

**Q: If `notify()` wakes a thread, but that thread's condition check still fails (via the `while` loop), what happens?** A: The thread simply calls `wait()` again, re-entering the wait set — effectively as if it had never been woken, from the perspective of program state (though this does have a small, real cost in terms of the wasted wake/re-acquire/re-check cycle).

**Q: Is `notify()` deprecated or discouraged by the JDK itself?** A: It's not deprecated, but it's widely considered, in practice, a dangerous, narrow-use-case tool compared to the safer `notifyAll()` default — most style guides and experienced practitioners recommend `notifyAll()` unless `notify()`'s safety conditions are explicitly, carefully verified.

---

## Revision Checklist

- [ ] I understand that `notify()` wakes exactly one, arbitrarily-chosen thread.
- [ ] I can state the two conditions required for `notify()` to be safe to use.
- [ ] I understand how misusing `notify()` with mixed waiter types can cause an indefinite stall.
- [ ] I understand that a "lost" notification (no one waiting) has no lasting effect.
- [ ] I know why `notifyAll()` is the generally recommended default.

---

## Flashcards

**Q:** How many threads does `notify()` wake? **A:** Exactly one, chosen arbitrarily by the JVM.

**Q:** What are the two conditions required for `notify()` to be safe? **A:** All waiters share the same wait condition, AND waking just one thread is sufficient for correct progress.

**Q:** What happens if `notify()` is called when no thread is waiting? **A:** Nothing — the notification is simply lost, with no lasting effect.

**Q:** Does `notify()` guarantee any particular thread (e.g., longest-waiting) gets woken? **A:** No — there's no ordering or fairness guarantee at all.

**Q:** What's the generally recommended default between `notify()` and `notifyAll()`? **A:** `notifyAll()`, unless `notify()`'s safety conditions are explicitly and carefully verified.

---

## References

- Oracle Java Tutorials — Guarded Blocks: https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html
- `java.lang.Object` API Documentation (`notify`, `notifyAll`, `wait`)
- JLS (Java Language Specification) — Chapter 17: Threads and Locks