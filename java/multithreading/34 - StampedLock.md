# StampedLock

## Objective

By the end of this note, you will understand:

- What `StampedLock` is and the specific limitation of `ReadWriteLock` it addresses
- The concept of "optimistic reading" — its most distinctive feature
- How to correctly validate an optimistic read, and when to fall back to a full read lock
- The trade-offs that make `StampedLock` more performant but also more complex and dangerous to misuse

This note builds directly on [[33 - ReadWriteLock]], addressing its main weakness: even a `ReadWriteLock`'s read lock has some overhead and can still be blocked by writers.

---

## Why should I learn this?

`StampedLock`, introduced in Java 8, was designed specifically to address a subtle limitation of `ReentrantReadWriteLock`: in read-heavy, low-write scenarios, even acquiring a read lock has some inherent overhead, and a long-held read lock can still delay a writer indefinitely. `StampedLock`'s **optimistic read** mode offers a way to read shared data **without ever blocking a writer at all**, trading some complexity for a meaningful performance gain in the right scenarios. It's less commonly used than `ReentrantLock`/`ReadWriteLock`, but understanding it demonstrates depth and is increasingly relevant in performance-sensitive modern Java code.

---

## Beginner Explanation

Imagine you want to quickly glance at a whiteboard to note down a number, without formally "signing out" any kind of access permission first. You just look, jot down what you saw, and move on. But there's a risk: what if someone was in the middle of erasing and rewriting the board exactly while you were glancing? Your noted-down number might be garbage — half of the old number, half of the new one.

The solution: after you've noted down your "optimistic" glance, you **quickly double check**: "was the board being edited at any point while I was looking?" If the answer is no, great — your reading was valid. If the answer is yes, you throw away what you noted and do a **proper, careful read** instead (formally waiting your turn if someone's currently editing).

This "glance first, validate after, fall back to a careful read only if needed" strategy is exactly what `StampedLock`'s **optimistic read** mode implements.

---

## Real World Analogy

**Airport Departure Board Analogy**

Imagine glancing up at an airport's departure board to check your gate number, without waiting in any kind of queue or asking permission — you just read what's currently displayed. Occasionally, the board might be in the middle of refreshing (flickering) exactly as you glance, and you might catch a garbled, half-updated value.

The safeguard: right after you glance, you quickly check "did the board flicker/update at any point while I was looking?" (This is like checking a version stamp.) If it didn't, you can trust what you saw. If it did, you glance again (or wait for a stable moment) to get a reliable reading. This is much faster than formally "signing out" a viewing permit every single time (like acquiring a full read lock), especially when board updates are rare.

---

## Technical Explanation

### The Three Modes of `StampedLock`

`StampedLock` supports three distinct locking modes:

1. **Write lock** — exclusive, like a normal write lock.
2. **Pessimistic read lock** — similar to `ReadWriteLock`'s read lock (multiple readers allowed, blocks writers).
3. **Optimistic read** — a genuinely new mode: doesn't block anything at all, but requires manual validation afterward.

### Using the Write Lock

```java
StampedLock lock = new StampedLock();

long stamp = lock.writeLock(); // blocks until acquired, like ReentrantLock
try {
    // update shared state
} finally {
    lock.unlockWrite(stamp); // must pass the SAME stamp back
}
```

Notice: `StampedLock` methods return a **`long` stamp**, which must be passed back to the corresponding unlock method — this stamp-based API is the most distinctive syntactic feature of `StampedLock`.

### Using the Pessimistic Read Lock

```java
long stamp = lock.readLock();
try {
    // read shared state
} finally {
    lock.unlockRead(stamp);
}
```

### Using Optimistic Read — The Key Feature

```java
long stamp = lock.tryOptimisticRead(); // NEVER blocks, returns immediately
int localX = x; // read shared state WITHOUT any lock held
int localY = y;

if (!lock.validate(stamp)) {
    // a write occurred during our read -- our local values may be inconsistent!
    // Fall back to a full, safe read lock:
    stamp = lock.readLock();
    try {
        localX = x;
        localY = y;
    } finally {
        lock.unlockRead(stamp);
    }
}

// safely use localX, localY here
```

**Critical detail**: `tryOptimisticRead()` does **not** actually acquire any lock at all — it just returns a stamp representing the lock's current version. You then read the shared data **completely unprotected**, and **afterward**, call `lock.validate(stamp)` to check: "did any write happen between when I got this stamp and now?" If validation fails, your just-read values might be corrupted/inconsistent, and you must fall back to a proper (pessimistic) read lock.

### Why This Is Faster

Since `tryOptimisticRead()` never blocks and never requires the CPU-level coordination a full lock acquisition needs, it's extremely cheap — especially valuable when writes are rare, since validation will succeed almost every time, letting you skip the more expensive locking machinery entirely in the common case.

---

## Internal Working

Step-by-step of an optimistic read:

1. The thread calls `tryOptimisticRead()`, receiving a stamp representing the lock's current "version" (essentially, has any write happened yet).
2. The thread reads shared data directly, with **no** lock held and **no** blocking of any writer that might come along.
3. **If a writer acquires the write lock during this window**, the lock's internal version changes.
4. The thread finishes its reads, then calls `lock.validate(stamp)`.
5. `validate()` checks if the version has changed since the stamp was issued — if unchanged, the reads were safe (no writer interfered); if changed, the reads may be corrupted, and the thread must retry with a proper `readLock()`.

---

## Memory Diagram

```
   StampedLock internal state
   -----------------------------
   | version stamp: 42          |
   -----------------------------

   Thread: tryOptimisticRead() -> gets stamp=42
   Thread: reads x, y (UNPROTECTED, no lock held)

   [MEANWHILE, possibly:]
   Writer: writeLock() -> acquires, updates x,y, version becomes 43
   Writer: unlockWrite()

   Thread: validate(42) -> FALSE (version is now 43, not 42)
   Thread: must retry with a real readLock()
```

---

## Execution Flow

```
Thread calls tryOptimisticRead() -- NEVER blocks
        |
        v
Reads shared data directly (no lock, no blocking anyone)
        |
        v
Calls lock.validate(stamp)
        |
   -----------------------------
   |                           |
 valid (no writer            invalid (a writer
 interfered)                  DID interfere)
   |                           |
   v                           v
Use the read values      Fall back: acquire full
safely                    readLock(), re-read safely
```

---

## Code Example

```java
import java.util.concurrent.locks.StampedLock;

public class StampedLockDemo {

    static final StampedLock lock = new StampedLock();
    static int x = 0;
    static int y = 0;

    static void writeValues(int newX, int newY) {
        long stamp = lock.writeLock();
        try {
            x = newX;
            try { Thread.sleep(50); } catch (InterruptedException e) { Thread.currentThread().interrupt(); } // simulate work
            y = newY;
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    static int[] readValuesOptimistically() {
        long stamp = lock.tryOptimisticRead();
        int localX = x;
        int localY = y;

        if (!lock.validate(stamp)) {
            System.out.println(Thread.currentThread().getName() + ": optimistic read INVALID, falling back to readLock()");
            stamp = lock.readLock();
            try {
                localX = x;
                localY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        } else {
            System.out.println(Thread.currentThread().getName() + ": optimistic read VALID, no locking needed!");
        }
        return new int[]{localX, localY};
    }

    public static void main(String[] args) throws InterruptedException {

        Thread writer = new Thread(() -> writeValues(100, 200), "Writer");

        Thread reader1 = new Thread(() -> {
            int[] result = readValuesOptimistically();
            System.out.println("Reader-1 got: x=" + result[0] + ", y=" + result[1]);
        }, "Reader-1");

        writer.start();
        Thread.sleep(10); // ensure writer has started its write (mid-update)
        reader1.start();  // likely to race with the writer's in-progress update

        writer.join();
        reader1.join();
    }
}
```

### Line-by-line explanation

1. `writeValues()` updates `x` then `y`, with a deliberate 50ms delay in between — simulating a "slow" write that leaves an inconsistent window if read during it.
2. `readValuesOptimistically()` uses `tryOptimisticRead()` to read `x` and `y` **without any lock**, then validates.
3. Because `reader1` starts **while** `writer` is likely mid-update (started just 10ms after the writer began its slow write), its optimistic read is likely to be **invalidated**, triggering the fallback to a proper `readLock()` — demonstrating the validation mechanism catching a genuinely risky read window.
4. Running this multiple times may show either outcome depending on exact timing, illustrating that `StampedLock`'s optimistic path is a **probabilistic fast path**, with a guaranteed-safe fallback always available.

---

## Dry Run

**Possible Output (optimistic read caught mid-write):**

```
Reader-1: optimistic read INVALID, falling back to readLock()
Reader-1 got: x=100, y=200
```

**Possible Output (optimistic read happened to complete before/after the write window):**

```
Reader-1: optimistic read VALID, no locking needed!
Reader-1 got: x=0, y=0
```

(or `x=100, y=200`, if it read cleanly after the write completed)

---

## Common Mistakes

1. **Forgetting to call `validate()` after an optimistic read.**
    
    - Without validation, you have **no guarantee** the values you read are consistent — this defeats the entire safety mechanism and can silently introduce corrupted reads into your program.
2. **Assuming optimistic reads are always successful (skipping the fallback path entirely).**
    
    - Under write-heavy conditions, optimistic reads can fail validation frequently — your code must **always** handle the fallback case properly, not just as a rare edge case.
3. **Using `StampedLock` for complex, general-purpose locking needs where its lack of reentrancy becomes a problem.**
    
    - Unlike `ReentrantLock` and `ReentrantReadWriteLock`, `StampedLock` is **not reentrant** — a thread that already holds a lock and tries to acquire it again (even the same mode) can deadlock against itself. This is a significant, easy-to-miss limitation.
4. **Mixing up stamps between different lock acquisitions.**
    
    - Each `lock()`/`tryOptimisticRead()` call returns a specific stamp that must be passed to the **matching** unlock/validate call — reusing or confusing stamps leads to incorrect behavior or exceptions.

---

## Production Usage

- **High-performance, read-dominant data structures** (e.g., certain custom caching or geometry/spatial-index implementations) use `StampedLock`'s optimistic mode to minimize overhead for the overwhelmingly common read case.
- **Java's own `java.time` internals** and some other JDK-internal high-performance code use `StampedLock` where the performance characteristics specifically justify its added complexity.
- **Most everyday application code does NOT use `StampedLock`** — it's a specialized tool reached for specifically when profiling has shown that `ReentrantReadWriteLock`'s overhead is a genuine bottleneck, and the non-reentrant, more error-prone API is deemed an acceptable trade-off for that specific hot path.

---

## Interview Questions

**Beginner:**

1. What is `StampedLock`, and what does it add beyond `ReadWriteLock`?
2. What is "optimistic reading"?
3. What must you always do after an optimistic read?

**Intermediate:** 4. What does `lock.validate(stamp)` check, precisely? 5. What should your code do if `validate()` returns `false`? 6. Is `StampedLock` reentrant?

**Advanced:** 7. Explain, step by step, how an optimistic read can end up reading inconsistent data, and how validation catches this. 8. Why is `StampedLock`'s non-reentrancy a significant practical limitation compared to `ReentrantReadWriteLock`?

**Production-Level:** 9. In what specific scenario would you consider replacing a `ReentrantReadWriteLock` with a `StampedLock`, and what would justify the added complexity/risk? 10. Why is `StampedLock` considered a more specialized, "expert-level" tool compared to `ReentrantLock` or `ReadWriteLock`?

---

## Follow-up Questions

- Does `tryOptimisticRead()` ever block? _(No — never; it always returns immediately with a stamp, with zero blocking, which is precisely its main performance advantage.)_
- Can `StampedLock` be converted between read and write modes, like `ReadWriteLock`'s downgrading? _(Yes — `StampedLock` supports its own conversion methods, like `tryConvertToWriteLock()`, offering similar but distinct capabilities from `ReadWriteLock`'s downgrading.)_
- What happens if a thread holding a write lock tries to acquire the write lock again (reentrantly)? _(It will block indefinitely, waiting for itself to release a lock it's already holding — since `StampedLock` is not reentrant, this causes a self-deadlock, unlike `ReentrantLock`/`ReentrantReadWriteLock`.)_

---

## Best Practices

- Always call `validate()` after every optimistic read, and always implement the pessimistic fallback path.
- Never assume `StampedLock` is reentrant — avoid nested lock acquisition patterns entirely with this class.
- Reach for `StampedLock` only after profiling has shown `ReentrantReadWriteLock` overhead is a genuine bottleneck for your specific, read-dominant hot path.
- Keep the optimistic read section (between `tryOptimisticRead()` and `validate()`) as short as possible, to minimize the window in which a writer could invalidate it.

---

## Performance Notes

- Optimistic reads are extremely cheap — no blocking, no CPU-level lock coordination — making `StampedLock` potentially faster than `ReentrantReadWriteLock` for read-dominant, low-contention scenarios.
- Under frequent writes, optimistic reads fail validation often, forcing frequent fallback to the pessimistic read lock — in such cases, the benefit over `ReentrantReadWriteLock` shrinks or disappears, and the added complexity may not be worth it.
- `StampedLock`'s lack of reentrancy removes some of the internal bookkeeping overhead `ReentrantLock`/`ReentrantReadWriteLock` require, contributing to its generally lower baseline overhead.

---

## Comparison Table

|Aspect|`ReentrantReadWriteLock`|`StampedLock`|
|---|---|---|
|Optimistic (non-blocking) read mode|No|Yes|
|Reentrant|Yes|No|
|Supports `Condition` objects|Yes|No|
|API complexity|Moderate|Higher (stamp-based, manual validation)|
|Best for|General read-heavy needs|Highly performance-critical, read-dominant hot paths|

---

## Summary

- `StampedLock` extends the read/write lock concept with a third mode: **optimistic reading**, which never blocks and requires manual validation afterward.
- If validation fails (a writer interfered), the code must fall back to a proper, blocking read lock.
- `StampedLock` is **not reentrant** — a significant limitation compared to `ReentrantLock`/`ReentrantReadWriteLock`, making it easier to misuse.
- It's best reserved for performance-critical, read-dominant scenarios where profiling has specifically justified its added complexity — not a default choice for general-purpose locking.

---

## Cheat Sheet

|Method|Behavior|
|---|---|
|`writeLock()`|Blocks until exclusive access acquired; returns a stamp|
|`readLock()`|Pessimistic read lock, like `ReadWriteLock`'s read lock|
|`tryOptimisticRead()`|Never blocks; returns a stamp representing current version|
|`validate(stamp)`|Checks if a write occurred since the stamp was issued|
|`unlockWrite(stamp)` / `unlockRead(stamp)`|Must pass the matching stamp|

---

## Keywords

StampedLock, Optimistic Read, Pessimistic Read, validate, Non-Reentrant, Stamp-Based API

---

## Related Notes

[[33 - ReadWriteLock]] [[32 - ReentrantLock]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Implement a simple shared pair of values protected by `StampedLock`, using the write lock for updates and optimistic reads for access.
2. Force a validation failure by reading during an in-progress write (as in this note's demo), and confirm the fallback path executes.

**Medium:** 3. Compare, via simple timing, an optimistic-read-based implementation against a `ReentrantReadWriteLock`-based implementation under a read-heavy, low-write workload. 4. Deliberately attempt reentrant lock acquisition with `StampedLock` (a thread acquiring the write lock twice) and observe the resulting deadlock (be ready to interrupt/stop the program).

**Hard:** 5. Implement a small, thread-safe 2D point class (`x`, `y` coordinates) using `StampedLock`'s optimistic read pattern, with a benchmark comparing it against both `synchronized` and `ReentrantReadWriteLock` versions under varying read/write ratios.

---

## Mini Project

**Project: Optimistic Read Success Rate Analyzer**

Build a small console application that:

1. Implements a shared data structure protected by `StampedLock`, with configurable read and write thread counts and write frequency.
2. Tracks, across many optimistic read attempts, what percentage succeed validation versus fall back to the pessimistic read lock.
3. Runs this experiment under several different write frequencies (rare, occasional, frequent) and reports the optimistic-read success rate for each.
4. Includes a brief written conclusion on when optimistic reads are worth using, based on the observed success rates.

**Goal:** Build empirical, hands-on intuition for exactly when `StampedLock`'s optimistic mode pays off — directly tying the "read-dominant" theoretical guidance to measured data.

---

## Common Interview Trap

**Trap Question:** _"Since `tryOptimisticRead()` never blocks and is very fast, can I skip the `validate()` call to make my code even faster?"_

The trap is treating `validate()` as an optional performance overhead rather than the essential safety mechanism it actually is. The correct answer: **no** — skipping `validate()` completely defeats the purpose of the optimistic read pattern. Without validation, you have **zero guarantee** that the values you read are consistent — you could easily read a half-written, corrupted combination of old and new values if a writer happened to be updating the data during your unprotected read. `validate()` is not an optional performance tax; it's the entire safety net that makes optimistic reading a legitimate, correct technique rather than a dangerous, un-synchronized data race.

---

## Frequently Asked Questions

**Q: Is `StampedLock` a drop-in replacement for `ReentrantReadWriteLock`?** A: No — due to its lack of reentrancy and lack of `Condition` support, it requires careful review of existing code before swapping it in; it's better thought of as a specialized alternative for specific hot paths, not a universal upgrade.

**Q: Can `StampedLock`'s pessimistic read and write locks be used without ever touching the optimistic mode?** A: Yes — you can use `StampedLock` purely in its pessimistic (blocking) modes, similar to `ReentrantReadWriteLock`, if you don't need the optimistic read capability; though at that point, `ReentrantReadWriteLock` might be simpler unless you specifically need `StampedLock`'s other characteristics.

**Q: What happens if `tryOptimisticRead()` is called while a writer currently holds the write lock?** A: It still returns a stamp immediately (never blocks), but that stamp will correctly fail validation once checked, since a write is in progress — triggering the appropriate fallback.

**Q: Does `StampedLock` support fairness like `ReentrantLock`?** A: `StampedLock` does not offer the same explicit fairness constructor parameter as `ReentrantLock`/`ReentrantReadWriteLock` — its internal scheduling behavior is simpler and geared toward maximizing throughput for its specific intended use cases.

---

## Revision Checklist

- [ ] I understand what "optimistic reading" means and why it's `StampedLock`'s signature feature.
- [ ] I know that `validate()` must always be called and handled properly after an optimistic read.
- [ ] I understand that `StampedLock` is NOT reentrant, unlike `ReentrantLock`/`ReentrantReadWriteLock`.
- [ ] I can explain when `StampedLock` is worth its added complexity versus when it isn't.
- [ ] I understand the stamp-based API pattern (matching stamps to unlock/validate calls).

---

## Flashcards

**Q:** What is `StampedLock`'s signature feature beyond `ReadWriteLock`? **A:** Optimistic reading — a non-blocking read mode requiring manual validation afterward.

**Q:** What must you always do after an optimistic read? **A:** Call `lock.validate(stamp)` and fall back to a proper read lock if it fails.

**Q:** Is `StampedLock` reentrant? **A:** No — attempting reentrant acquisition can cause a self-deadlock.

**Q:** When is `StampedLock` worth using over `ReentrantReadWriteLock`? **A:** In performance-critical, read-dominant hot paths, specifically after profiling justifies the added complexity.

**Q:** Does `tryOptimisticRead()` ever block? **A:** No — it always returns immediately.

---

## References

- Oracle Java Tutorials — Lock Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html
- `java.util.concurrent.locks.StampedLock` API Documentation