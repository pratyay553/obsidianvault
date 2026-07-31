# ReentrantLock

## Objective

By the end of this note, you will understand:

- What `ReentrantLock` is and how it relates to the intrinsic locks covered in [[24 - Intrinsic Lock]]
- The specific capabilities it offers beyond `synchronized`: try-lock, timed lock, interruptible lock, and fairness
- The mandatory `try/finally` pattern and why it's non-negotiable
- When to reach for `ReentrantLock` instead of `synchronized` in real code

This note opens **Section 5: Advanced Locks**, building directly on the intrinsic-vs-explicit lock preview from [[24 - Intrinsic Lock]].

---

## Why should I learn this?

`synchronized` is simple and safe, but it has hard limits: you can't interrupt a thread stuck waiting for a lock, you can't attempt to acquire a lock without potentially blocking forever, and you can't set a timeout. `ReentrantLock`, part of `java.util.concurrent.locks` since Java 5, removes these limits — at the cost of requiring you to manage the lock's lifecycle yourself. Understanding exactly when this trade-off is worth it is a mark of genuine concurrency maturity, and a very common interview topic.

---

## Beginner Explanation

`ReentrantLock` is a standalone **object** that behaves like a lock — you explicitly call `.lock()` to acquire it and `.unlock()` to release it, rather than relying on Java's automatic `synchronized` block entry/exit.

Think of the difference between an **automatic revolving door** (like `synchronized` — it lets people in and out with built-in safety, no way to misuse it) and a **manual door with a handle** (like `ReentrantLock` — you must remember to open it and, critically, remember to close it yourself). The manual door gives you more control (you can decide to peek through first without committing to enter, or give up and walk away if it's taking too long) — but you bear the responsibility of not leaving it stuck open.

---

## Real World Analogy

**Hospital Analogy**

Imagine a hospital's operating room, protected by two different systems:

- **`synchronized` (automatic system)**: the door automatically locks when a surgeon enters and automatically unlocks when they leave — completely foolproof, no way to forget.
- **`ReentrantLock` (manual system)**: a nurse manually checks out a key from the front desk, uses it to enter, and must manually return it when done. This system additionally allows the nurse to: check if the room is available **without waiting** (`tryLock()`), wait only up to a specific time before giving up (`tryLock(timeout, unit)`), or be paged away (interrupted) if an emergency arises elsewhere while waiting for the room (`lockInterruptibly()`) — capabilities the fully-automatic door simply cannot offer.

---

## Technical Explanation

### Basic Usage — The Mandatory Pattern

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

Lock lock = new ReentrantLock();

lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // MUST be in finally, or a lock leak occurs on exception
}
```

**This `try/finally` pattern is non-negotiable.** Unlike `synchronized`, the JVM provides **no automatic guarantee** that `ReentrantLock.unlock()` gets called — if an exception occurs inside the critical section and you forget the `finally` block, the lock remains held **forever**, permanently blocking every other thread that needs it (see [[24 - Intrinsic Lock]] for this exact risk highlighted earlier).

### Capability 1: `tryLock()` — Non-Blocking Attempt

```java
if (lock.tryLock()) {
    try {
        // acquired the lock without waiting at all
    } finally {
        lock.unlock();
    }
} else {
    // could not acquire immediately -- do something else instead of waiting
    System.out.println("Lock was busy, skipping this attempt.");
}
```

`tryLock()` attempts to acquire the lock and returns **immediately** — `true` if successful, `false` if the lock was already held by another thread. This has **no equivalent** in `synchronized`, which always blocks indefinitely if the lock is unavailable.

### Capability 2: `tryLock(timeout, unit)` — Bounded Waiting

```java
if (lock.tryLock(2, TimeUnit.SECONDS)) {
    try {
        // acquired within 2 seconds
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not acquire lock within 2 seconds, giving up.");
}
```

This waits **up to** the specified duration, then gives up — useful for avoiding indefinite blocking in scenarios where waiting forever is unacceptable (e.g., a request-handling thread that shouldn't hang the entire request pipeline).

### Capability 3: `lockInterruptibly()` — Interruptible Waiting

```java
try {
    lock.lockInterruptibly(); // can be interrupted while WAITING for the lock
    try {
        // critical section
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    // handle cancellation
}
```

As covered in [[21 - Interrupt()]], a thread `BLOCKED` waiting to enter a `synchronized` block **cannot** be interrupted out of that wait. `lockInterruptibly()` solves exactly this limitation — a thread waiting via this method **can** be woken up early by another thread calling `.interrupt()` on it, throwing `InterruptedException` instead of continuing to wait.

### Capability 4: Fairness

```java
Lock fairLock = new ReentrantLock(true); // fair mode
```

By default (`new ReentrantLock()`, equivalent to `new ReentrantLock(false)`), `ReentrantLock` does **not** guarantee strict first-come-first-served ordering among waiting threads (similar to `synchronized`'s default behavior — see [[38 - Starvation]]). Passing `true` to the constructor enables **fair mode**, where the longest-waiting thread is granted the lock next — at the cost of somewhat reduced throughput, since strict fairness requires more coordination overhead.

### Reentrancy — Same as Intrinsic Locks

As the name suggests, `ReentrantLock` supports the same reentrancy behavior as intrinsic locks (see [[23 - Monitor Lock]]):

```java
lock.lock();
try {
    lock.lock(); // same thread, reentrant -- succeeds immediately, hold count = 2
    try {
        // ...
    } finally {
        lock.unlock(); // hold count = 1
    }
} finally {
    lock.unlock(); // hold count = 0, fully released
}
```

**Critical detail**: the number of `.unlock()` calls must exactly match the number of `.lock()` calls for the lock to actually be released — an internal hold count tracks this.

---

## Internal Working

Step-by-step of what happens with `lock.lock()`:

1. The thread checks: is the lock currently unowned, or owned by this same thread already (reentrant case)?
2. If unowned or reentrant: the thread becomes/remains the owner, and the internal hold count is incremented.
3. If owned by a **different** thread: the calling thread is queued (internally, typically using a structure like `AbstractQueuedSynchronizer`, the underlying framework `ReentrantLock` is built on) and blocks until the lock becomes available.
4. Upon `.unlock()`, the hold count is decremented; if it reaches zero, the lock is fully released, and the next queued thread (if any) is granted access according to the lock's fairness policy.

Unlike `synchronized`, none of this acquire/release bookkeeping is automatic — every step depends on your code correctly calling `.lock()` and `.unlock()` in the right places.

---

## Memory Diagram

```
        ReentrantLock object (a SEPARATE object from any "protected" data)
   -----------------------------------------------
   |  owner: Thread-A (or null)                    |
   |  hold count: 2 (if Thread-A reentered once)   |
   |  queue: [Thread-B, Thread-C waiting]          |
   |  fair: true/false                              |
   -----------------------------------------------
```

---

## Execution Flow

```
Thread calls lock.lock()
        |
        v
Is lock free, or owned by THIS thread already?
        |
   -----------------------------
   |                           |
  YES                         NO (owned by another thread)
   |                           |
   v                           v
Acquire/reenter,          Queue and BLOCK until
hold count++               lock becomes available
   |                           |
   v                           |
Execute critical section       |
   |                           |
   v                           |
finally { lock.unlock() }      |
hold count--                   |
   |                           |
   v                           |
If hold count == 0,            |
release lock, notify next      |
queued thread (per fairness)  -+
```

---

## Code Example

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {

    static final Lock lock = new ReentrantLock();
    static int sharedResource = 0;

    public static void main(String[] args) throws InterruptedException {

        // --- Demo 1: standard lock/unlock pattern ---
        Runnable safeIncrement = () -> {
            lock.lock();
            try {
                sharedResource++;
            } finally {
                lock.unlock();
            }
        };

        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 10_000; j++) safeIncrement.run();
            });
        }
        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();
        System.out.println("sharedResource: " + sharedResource + " (expected 100000)");

        // --- Demo 2: tryLock() with a timeout ---
        lock.lock(); // main thread deliberately holds the lock
        Thread attempt = new Thread(() -> {
            try {
                boolean acquired = lock.tryLock(1, TimeUnit.SECONDS);
                if (acquired) {
                    try {
                        System.out.println("Somehow acquired the lock.");
                    } finally {
                        lock.unlock();
                    }
                } else {
                    System.out.println("Could not acquire lock within 1 second, as expected.");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        attempt.start();
        attempt.join();
        lock.unlock(); // main thread releases it afterward
    }
}
```

### Line-by-line explanation

1. `safeIncrement` demonstrates the standard, mandatory `lock()`/`try`/`finally { unlock() }` pattern — 10 threads × 10,000 increments reliably produces exactly 100,000.
2. In Demo 2, the main thread deliberately holds `lock` before starting `attempt`, so `attempt`'s `tryLock(1, TimeUnit.SECONDS)` call is guaranteed to time out after 1 second, printing the expected "could not acquire" message — directly demonstrating a capability `synchronized` simply does not have.

---

## Dry Run

**Demo 1:**

```
sharedResource: 100000 (expected 100000)
```

**Demo 2:**

```
Could not acquire lock within 1 second, as expected.
```

---

## Common Mistakes

1. **Forgetting the `finally` block, or placing `.unlock()` outside it.**
    
    - If an exception occurs between `.lock()` and `.unlock()`, and `.unlock()` isn't in a `finally` block, the lock is **never released**, permanently blocking every other thread needing it — this is the single most dangerous, common `ReentrantLock` mistake.
2. **Calling `.unlock()` without a matching prior `.lock()` call (e.g., from a different thread than the one that acquired it).**
    
    - This throws `IllegalMonitorStateException` — only the thread that acquired a `ReentrantLock` may unlock it (this matches intrinsic lock behavior).
3. **Mismatched lock/unlock counts when using reentrancy.**
    
    - If a thread calls `.lock()` twice (reentrant) but only calls `.unlock()` once, the lock remains held by that thread (hold count still 1), unexpectedly blocking other threads.
4. **Using `ReentrantLock` reflexively everywhere, even when `synchronized` would be simpler and sufficient.**
    
    - Reach for `ReentrantLock` specifically when you need one of its extra capabilities (try-lock, timeout, interruptibility, fairness) — otherwise, `synchronized`'s automatic safety is generally the better default.

---

## Production Usage

- **Connection pool implementations** commonly use `tryLock(timeout, unit)` to avoid indefinitely blocking a thread waiting for a pooled resource, instead failing fast with a clear error after a bounded wait.
- **Cancellable, long-running operations** use `lockInterruptibly()` so that a user-cancelled or timed-out operation can be promptly abandoned even if it's currently waiting for a lock, rather than being stuck.
- **Systems requiring fairness guarantees** (e.g., avoiding thread starvation in specific high-stakes scheduling scenarios) use the fair-mode constructor, accepting the throughput trade-off for predictable ordering.
- **`ReadWriteLock`** (see [[33 - ReadWriteLock]]) and other advanced coordination primitives are built using the same `java.util.concurrent.locks` foundation as `ReentrantLock`.

---

## Interview Questions

**Beginner:**

1. What is `ReentrantLock`, and how does it differ from `synchronized`?
2. What must you always do after calling `lock.lock()`, without exception?
3. What does `tryLock()` do differently from `lock()`?

**Intermediate:** 4. Why can't a thread `BLOCKED` on a `synchronized` lock be interrupted, but a thread waiting via `lockInterruptibly()` can be? 5. What does the `fair` constructor parameter do, and what's the trade-off? 6. What happens if you call `.unlock()` more times than you called `.lock()`?

**Advanced:** 7. Why is `ReentrantLock`'s automatic release guarantee weaker than `synchronized`'s, and how do you compensate for this in your own code? 8. Describe a scenario where `tryLock(timeout, unit)` is clearly the better choice over standard blocking `lock()`.

**Production-Level:** 9. Why might a connection pool implementation specifically choose `ReentrantLock` with a timeout over a `synchronized` block? 10. What risk does `ReentrantLock`'s manual lifecycle management introduce that `synchronized` fundamentally cannot have, and how do teams mitigate this risk (code review, static analysis, conventions)?

---

## Follow-up Questions

- Can two different `ReentrantLock` instances be used to protect two genuinely independent pieces of shared state, with no interaction between them? _(Yes — exactly like using two separate intrinsic locks/monitors, this is a valid, common pattern known as lock splitting; see [[27 - Critical Section]].)_
- Does `ReentrantLock` support `wait()`/`notify()`-style coordination? _(Not directly via `Object`'s methods, but via its own `Condition` objects, created through `lock.newCondition()` — full detail in [[35 - Condition]].)_
- Is `ReentrantLock` slower than `synchronized`? _(Historically there were more noticeable differences; in modern JVMs, both are highly optimized, and performance is often comparable for the uncontended case — the choice is primarily about capability needs, not raw speed.)_

---

## Best Practices

- **Always** use `try/finally` with `ReentrantLock` — no exceptions to this rule, ever.
- Default to `synchronized` for simple cases; reach for `ReentrantLock` specifically when you need try-lock, timeout, interruptibility, or fairness.
- Ensure lock/unlock calls are balanced, especially when reentrancy is involved.
- Consider using fair mode only when you have a genuine, demonstrated need for strict ordering — it comes with a real throughput cost.

---

## Performance Notes

- Uncontended `ReentrantLock` acquisition is generally comparable in cost to uncontended `synchronized` in modern JVMs.
- Fair-mode `ReentrantLock` has meaningfully higher overhead than non-fair mode, due to the additional bookkeeping required to enforce strict ordering — use it deliberately, not by default.
- `tryLock()`/`tryLock(timeout, unit)` can improve overall system responsiveness by avoiding indefinite blocking, at the cost of needing to handle the "didn't acquire" case gracefully in your application logic.

---

## Comparison Table

_(See the detailed capability comparison table in [[24 - Intrinsic Lock]] for the full intrinsic vs. explicit lock breakdown.)_

|Capability|`synchronized`|`ReentrantLock`|
|---|---|---|
|Automatic release|Yes|No (`try/finally` required)|
|Non-blocking attempt (`tryLock`)|No|Yes|
|Timeout-bounded wait|No|Yes|
|Interruptible wait|No|Yes|
|Fairness option|No|Yes|
|Reentrant|Yes|Yes|

---

## Summary

- `ReentrantLock` is an explicit lock class offering capabilities `synchronized` cannot: non-blocking attempts (`tryLock()`), timeout-bounded waiting, interruptible waiting (`lockInterruptibly()`), and optional fairness.
- Unlike `synchronized`, it does **not** automatically release on exception — the `try/finally` pattern is mandatory, every single time.
- It supports the same reentrancy behavior as intrinsic locks, tracked via an internal hold count.
- Reach for it specifically when you need one of its extra capabilities; otherwise, `synchronized` remains the simpler, safer default for straightforward critical sections.

---

## Cheat Sheet

|Method|Behavior|
|---|---|
|`lock()`|Blocks until acquired (like `synchronized`, but manual release)|
|`tryLock()`|Attempts immediately, returns `true`/`false`, never blocks|
|`tryLock(timeout, unit)`|Waits up to timeout, then gives up|
|`lockInterruptibly()`|Waits, but can be interrupted early|
|`unlock()`|MUST be called in `finally`|
|`new ReentrantLock(true)`|Enables fair mode|

---

## Keywords

ReentrantLock, tryLock, lockInterruptibly, Fairness, Hold Count, try/finally, java.util.concurrent.locks

---

## Related Notes

[[24 - Intrinsic Lock]] [[21 - Interrupt()]] [[33 - ReadWriteLock]] [[34 - StampedLock]] [[35 - Condition]] [[38 - Starvation]]

---

## Practice Exercises

**Easy:**

1. Implement a thread-safe counter using `ReentrantLock`, with correct `try/finally` usage, and confirm correctness under concurrent access.
2. Demonstrate `tryLock()`'s immediate, non-blocking behavior by having one thread hold the lock while another attempts `tryLock()` and observes `false`.

**Medium:** 3. Demonstrate `lockInterruptibly()` by having a thread wait for a lock held by another thread, then interrupting the waiting thread and confirming it wakes up via `InterruptedException` rather than continuing to wait. 4. Compare fair mode versus non-fair mode empirically: run many threads competing for a lock under both settings, and observe (via logging acquisition order) whether fair mode produces more strictly ordered acquisition.

**Hard:** 5. Build a small connection-pool-like resource manager using `tryLock(timeout, unit)` to bound how long a caller waits for a pooled resource before receiving a "pool exhausted" error instead of blocking indefinitely.

---

## Mini Project

**Project: Resilient Resource Pool**

Build a small console application that:

1. Implements a simple pool of N "resources" (e.g., simulated database connections), protected by a `ReentrantLock`.
2. Provides an `acquire(timeout)` method using `tryLock(timeout, unit)` combined with a condition/count check, returning either a resource or a "timed out" result.
3. Simulates many concurrent client threads requesting resources with varying timeouts, some successfully acquiring, others timing out gracefully.
4. Logs a summary of successful acquisitions versus timeouts, demonstrating graceful degradation under contention rather than indefinite blocking.

**Goal:** Apply `ReentrantLock`'s advanced capabilities to a realistic, production-flavored resource management scenario — reinforcing why these capabilities matter beyond the basic mechanics.

---

## Common Interview Trap

**Trap Question:** _"Since `ReentrantLock` offers strictly more capabilities than `synchronized` (try-lock, timeout, interruptibility, fairness), should I just always use `ReentrantLock` instead of `synchronized` by default?"_

The trap is assuming "more capable" automatically means "better default choice." The correct, more experienced answer: **no** — `synchronized`'s automatic acquire/release (guaranteed even on exceptions) makes it fundamentally **safer by default**, since it's impossible to accidentally "forget" to release it. `ReentrantLock` requires disciplined, correct manual management (`try/finally`, matched lock/unlock counts) — a single missed `finally` block can cause a permanent lock leak that `synchronized` simply cannot produce. The correct engineering approach is to default to `synchronized` for typical critical sections, and deliberately reach for `ReentrantLock` **only** when you have a specific, genuine need for one of its extra capabilities — not as a blanket default "because it can do more."

---

## Frequently Asked Questions

**Q: Does `ReentrantLock` implement the `Lock` interface, or is `Lock` a separate thing?** A: `ReentrantLock` is the primary, standard implementation of the `java.util.concurrent.locks.Lock` interface — `Lock` defines the contract (`lock()`, `unlock()`, `tryLock()`, etc.), and `ReentrantLock` is the concrete class you typically instantiate.

**Q: Can you check whether a `ReentrantLock` is currently held, without trying to acquire it?** A: Yes — methods like `isLocked()` and `getHoldCount()` provide introspection, though these are typically used for monitoring/debugging rather than as part of core application logic (since checking and then acting isn't atomic — see [[26 - Race Condition]]).

**Q: Is fairness guaranteed to be perfectly strict FIFO order with `new ReentrantLock(true)`?** A: It provides a strong, well-documented approximation of FIFO fairness among threads that have been waiting, though the JDK documentation itself notes that absolute, guaranteed ordering under all circumstances is a nuanced topic — in practice, fair mode is reliably close to first-come-first-served for typical use cases.

**Q: Does `ReentrantLock` work correctly with `try-with-resources` (Java's automatic resource management syntax)?** A: Not directly out of the box, since `Lock` doesn't implement `AutoCloseable` in the standard JDK — some codebases wrap `ReentrantLock` usage in a custom `AutoCloseable` helper to get similar `try-with-resources` convenience, though the standard, universally understood pattern remains explicit `try/finally`.

---

## Revision Checklist

- [ ] I understand why the `try/finally` pattern is mandatory with `ReentrantLock`.
- [ ] I can list and explain all four extra capabilities `ReentrantLock` provides over `synchronized`.
- [ ] I understand `ReentrantLock`'s reentrancy and hold-count behavior.
- [ ] I know when to default to `synchronized` versus deliberately choosing `ReentrantLock`.
- [ ] I understand the fairness trade-off (ordering guarantee vs. throughput cost).

---

## Flashcards

**Q:** What must you always do after calling `lock.lock()` on a `ReentrantLock`? **A:** Wrap the critical section in `try` and call `lock.unlock()` in a `finally` block.

**Q:** What does `tryLock()` do that `lock()` does not? **A:** It attempts to acquire the lock immediately, returning `true`/`false`, without blocking indefinitely.

**Q:** Can a thread `BLOCKED` on `synchronized` be interrupted out of its wait? **A:** No — but a thread waiting via `lockInterruptibly()` on a `ReentrantLock` can be.

**Q:** What does passing `true` to the `ReentrantLock` constructor do? **A:** Enables fair mode, granting the lock to the longest-waiting thread, at some throughput cost.

**Q:** Should `ReentrantLock` be the default choice over `synchronized` for simple critical sections? **A:** No — `synchronized` is safer by default (automatic release); use `ReentrantLock` only when you need its specific extra capabilities.

---

## References

- Oracle Java Tutorials — Lock Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html
- `java.util.concurrent.locks.Lock` API Documentation
- `java.util.concurrent.locks.ReentrantLock` API Documentation