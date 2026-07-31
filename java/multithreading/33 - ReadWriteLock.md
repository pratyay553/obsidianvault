# ReadWriteLock

## Objective

By the end of this note, you will understand:

- What `ReadWriteLock` is and the specific performance problem it solves
- The distinction between the read lock and write lock, and their concurrency rules
- How to use `ReentrantReadWriteLock` correctly
- When this pattern is worth the added complexity, and when it isn't

This note builds on [[32 - ReentrantLock]] and addresses a very common real-world scenario: data that's **read far more often than it's written**.

---

## Why should I learn this?

A plain `ReentrantLock` (or `synchronized`) only allows **one thread at a time**, period — even for pure read operations that don't modify anything. But in many real systems, reads vastly outnumber writes (think: a configuration object read by every request, but updated only occasionally). Forcing all those reads to fully serialize, one at a time, is a massive, unnecessary performance bottleneck. `ReadWriteLock` solves exactly this problem by allowing **multiple concurrent readers**, while still guaranteeing **exclusive access for writers**.

---

## Beginner Explanation

Imagine a library's reference section, where many people can **read** the same reference book simultaneously without any conflict — reading doesn't change the book, so there's no harm in many people doing it at once. But if someone needs to **update** the book (say, correct an error or annotate a page), everyone else needs to step back and let that one person work alone, since simultaneous reading and editing could show readers an inconsistent, half-edited page.

`ReadWriteLock` implements exactly this policy in code: **many simultaneous readers are fine, but a writer needs completely exclusive access** — no readers or other writers allowed while a write is happening.

---

## Real World Analogy

**Museum Analogy**

Imagine a museum exhibit hall: any number of visitors can walk through and **view** the exhibits at the same time — no conflict, since viewing doesn't change anything. But when the museum staff needs to **rearrange** an exhibit (update the display), they close that hall entirely — no visitors allowed in, and any current visitors must finish and leave, until the rearrangement is complete. Once done, the hall reopens for unlimited simultaneous viewing again.

This "many viewers OR one exclusive rearranger, never both at once" policy is precisely the `ReadWriteLock` model.

---

## Technical Explanation

### The `ReadWriteLock` Interface and `ReentrantReadWriteLock`

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
```

`ReadWriteLock` exposes **two separate `Lock` objects**: a read lock and a write lock, both derived from the same underlying `ReentrantReadWriteLock` instance, but with different concurrency rules.

### The Concurrency Rules

|Lock Held|Can another thread acquire the READ lock?|Can another thread acquire the WRITE lock?|
|---|---|---|
|No one holds anything|Yes|Yes|
|Read lock (by one or more threads)|Yes (multiple readers allowed)|No — must wait for all readers to finish|
|Write lock (by one thread)|No — must wait for the writer to finish|No — must wait for the writer to finish|

In short: **readers don't block other readers**, but **a writer blocks everyone** (both other writers and all readers), and **readers block a writer** from starting.

### Usage Pattern

```java
class SharedConfig {
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private Map<String, String> config = new HashMap<>();

    public String getValue(String key) {
        readLock.lock();
        try {
            return config.get(key);
        } finally {
            readLock.unlock();
        }
    }

    public void updateConfig(Map<String, String> newConfig) {
        writeLock.lock();
        try {
            config = newConfig;
        } finally {
            writeLock.unlock();
        }
    }
}
```

Just like `ReentrantLock`, both the read and write locks require the exact same `try/finally` discipline — there's no automatic release.

### Lock Downgrading (An Advanced, Specific Capability)

`ReentrantReadWriteLock` supports a specific pattern called **lock downgrading**: a thread holding the write lock can acquire the read lock **before** releasing the write lock, ensuring it retains at least read-level protection continuously:

```java
writeLock.lock();
try {
    // make updates
    readLock.lock(); // acquire read lock WHILE still holding write lock
} finally {
    writeLock.unlock(); // now downgraded to just holding the read lock
}
try {
    // continue reading with guaranteed consistency, now with read lock only
} finally {
    readLock.unlock();
}
```

**Important**: the reverse — **upgrading** from a read lock to a write lock while holding the read lock — is **not supported** and will cause a deadlock (the write lock will wait forever for the read lock, held by the same thread, to be released first).

---

## Internal Working

Step-by-step of how `ReentrantReadWriteLock` tracks state internally:

1. It maintains an internal count of how many threads currently hold the **read** lock, plus tracking whether the **write** lock is currently held (and by which thread, supporting write-lock reentrancy).
2. When a thread requests the **read** lock: if no writer currently holds the write lock, it's granted immediately (incrementing the reader count); if a writer holds it, the requesting thread blocks.
3. When a thread requests the **write** lock: it's only granted if **no** readers currently hold the read lock **and** no other writer holds the write lock; otherwise, it blocks until all current holders release.
4. Read lock releases decrement the reader count; write lock release fully frees the lock for the next waiting reader(s) or writer.

---

## Memory Diagram

```
        ReentrantReadWriteLock
   -----------------------------------
   |  reader count: 3                 |  <- 3 threads currently reading
   |  writer: null                    |
   -----------------------------------

   A 4th thread wants the READ lock -> granted immediately (readers don't block readers)

   A thread wants the WRITE lock -> must WAIT until all 3 (or 4) readers finish
```

---

## Execution Flow

```
Multiple threads call readLock.lock()
        |
        v
All granted concurrently (as long as no writer holds the lock)
        |
        v
A thread calls writeLock.lock()
        |
        v
Must wait until ALL current readers release their read locks
        |
        v
Writer granted EXCLUSIVE access (no readers, no other writers)
        |
        v
Writer calls writeLock.unlock()
        |
        v
Waiting readers (and/or the next writer) granted access
```

---

## Code Example

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;
import java.util.concurrent.locks.Lock;

public class ReadWriteLockDemo {

    static final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    static final Lock readLock = rwLock.readLock();
    static final Lock writeLock = rwLock.writeLock();
    static int sharedData = 0;

    static void readData(String readerName) {
        readLock.lock();
        try {
            System.out.println(readerName + " reading value: " + sharedData +
                    " at " + System.currentTimeMillis() % 100000);
            try { Thread.sleep(500); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        } finally {
            readLock.unlock();
        }
    }

    static void writeData(int newValue) {
        writeLock.lock();
        try {
            System.out.println("Writer updating value to " + newValue +
                    " at " + System.currentTimeMillis() % 100000);
            sharedData = newValue;
            try { Thread.sleep(500); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
        } finally {
            writeLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        Thread reader1 = new Thread(() -> readData("Reader-1"));
        Thread reader2 = new Thread(() -> readData("Reader-2"));
        Thread reader3 = new Thread(() -> readData("Reader-3"));
        Thread writer = new Thread(() -> writeData(99));

        reader1.start();
        reader2.start();
        reader3.start(); // all 3 readers should run CONCURRENTLY (overlapping timestamps)

        Thread.sleep(100);
        writer.start(); // should WAIT until all 3 readers finish

        reader1.join();
        reader2.join();
        reader3.join();
        writer.join();
    }
}
```

### Line-by-line explanation

1. Three reader threads all call `readData()` at nearly the same time — since readers don't block other readers, all three should print their "reading value" messages with **overlapping** timestamps, and all three `Thread.sleep(500)` calls happen **concurrently**.
2. The `writer` thread starts shortly after and attempts `writeData()` — because readers currently hold the read lock, the writer must **wait** until all three readers finish (each sleeping 500ms), only then proceeding with its own exclusive write.
3. This demonstrates the core behavior: readers run in parallel with each other, but a writer waits for all of them and then gets fully exclusive access.

---

## Dry Run

|Time|Reader-1|Reader-2|Reader-3|Writer|
|---|---|---|---|---|
|0ms|Acquires read lock, reading|Acquires read lock, reading|Acquires read lock, reading|Not started|
|100ms|(reading)|(reading)|(reading)|Starts, tries write lock, BLOCKED (readers active)|
|500ms|Releases read lock|Releases read lock|Releases read lock|Still blocked until ALL release|
|500ms+|(done)|(done)|(done)|Acquires write lock, updates value|

---

## Common Mistakes

1. **Using `ReadWriteLock` for data that's written just as often as (or more than) it's read.**
    
    - The overhead of managing two separate locks only pays off when reads significantly outnumber writes; for balanced or write-heavy workloads, a plain `ReentrantLock` (or `synchronized`) is simpler and often just as fast or faster.
2. **Attempting to upgrade from a read lock to a write lock while holding the read lock.**
    
    - This causes a deadlock — the write lock waits for the read lock (held by the same thread) to be released, which will never happen since the same thread is stuck waiting on the write lock. Only **downgrading** (write → read) is safely supported.
3. **Forgetting that a writer blocks readers too, not just other writers.**
    
    - Some beginners assume "read lock" means readers are always instantly available — but if a writer currently holds the write lock, even new readers must wait.
4. **Not using `try/finally` for both the read and write lock, exactly as required for any explicit lock.**
    

---

## Production Usage

- **Configuration objects** read by every request but updated only occasionally (e.g., feature flags, routing rules) are a textbook use case for `ReadWriteLock`.
- **In-memory caches** with far more read traffic than write/refresh traffic often use this pattern to maximize concurrent read throughput.
- **Metadata/schema caches** in database drivers or ORMs, read constantly during normal operation but updated rarely (e.g., on schema changes), commonly benefit from this pattern.
- **`ConcurrentHashMap`** itself achieves much of its high read concurrency using more sophisticated internal techniques (not a simple `ReadWriteLock`, but conceptually related in spirit) — for many general-purpose use cases, reaching for `ConcurrentHashMap` directly is preferred over hand-rolling a `ReadWriteLock`-protected `HashMap` (see [[52 - ConcurrentHashMap]]).

---

## Interview Questions

**Beginner:**

1. What problem does `ReadWriteLock` solve that a plain `ReentrantLock` does not?
2. Can multiple threads hold the read lock at the same time?
3. Can a reader and a writer hold their respective locks at the same time?

**Intermediate:** 4. What happens to new read-lock requests while a writer currently holds the write lock? 5. What is "lock downgrading," and why is it supported while "upgrading" is not? 6. In what scenario would `ReadWriteLock` NOT provide a meaningful benefit over a plain lock?

**Advanced:** 7. Explain, step by step, why attempting to upgrade from a read lock to a write lock (while still holding the read lock) causes a deadlock. 8. How does `ReentrantReadWriteLock` track its internal state to correctly enforce the reader/writer concurrency rules?

**Production-Level:** 9. Describe a real production scenario where `ReadWriteLock` would provide a meaningful throughput improvement, with rough read/write ratio reasoning. 10. Why might `ConcurrentHashMap` be preferred over a hand-rolled `ReadWriteLock`-protected `HashMap` for many general-purpose caching needs?

---

## Follow-up Questions

- Does `ReentrantReadWriteLock` support fairness, like `ReentrantLock`? _(Yes — it has a similar constructor parameter for fair mode, affecting the ordering of granted access between waiting readers and writers.)_
- Is the read lock itself reentrant? _(Yes — a thread already holding the read lock can acquire it again; similarly, the write lock is independently reentrant for a thread that already holds it.)_
- Can a thread holding the write lock also acquire the read lock without any special "downgrading" concern, since it already has exclusive access? _(Yes — this is exactly what lock downgrading is: safely acquiring the read lock while still holding the write lock, then releasing the write lock, ending up with just the read lock.)_

---

## Best Practices

- Use `ReadWriteLock` specifically when reads significantly outnumber writes and read operations can safely run concurrently.
- Never attempt to upgrade from read lock to write lock while holding the read lock — release the read lock first, then acquire the write lock (accepting that another thread could modify the data in between, if that matters for your logic).
- Use lock downgrading (write → read) when you need to make an update and then continue with guaranteed-consistent reads without a gap where another writer could sneak in.
- For simple general-purpose thread-safe map/cache needs, consider `ConcurrentHashMap` before hand-rolling a `ReadWriteLock`-based solution.

---

## Performance Notes

- `ReadWriteLock` provides real throughput benefits specifically for read-heavy workloads — the more reads outnumber writes, the bigger the potential gain from allowing concurrent reads.
- For write-heavy or balanced workloads, the added bookkeeping overhead of tracking readers/writers separately can make `ReadWriteLock` perform **worse** than a simple `ReentrantLock` — always consider your actual read/write ratio before choosing this pattern.

---

## Comparison Table

|Aspect|`ReentrantLock`|`ReentrantReadWriteLock`|
|---|---|---|
|Concurrent readers allowed?|No — one thread at a time, period|Yes|
|Best for|Balanced or write-heavy access|Read-heavy access|
|Complexity|Lower|Higher (two locks to manage)|
|Risk of misuse|Forgetting `unlock()`|Forgetting `unlock()`, PLUS read-to-write upgrade deadlock risk|

---

## Summary

- `ReadWriteLock` allows multiple concurrent readers while still guaranteeing a writer gets fully exclusive access (blocking both other writers and all readers).
- `ReentrantReadWriteLock` is the standard JDK implementation, exposing separate `readLock()` and `writeLock()` `Lock` objects.
- Both locks require the standard `try/finally` discipline for safe release.
- Lock downgrading (write → read) is safely supported; upgrading (read → write) is not, and will deadlock.
- This pattern is worth its added complexity specifically for read-heavy workloads — for balanced or write-heavy access, a simple `ReentrantLock` is often better.

---

## Cheat Sheet

|Scenario|Result|
|---|---|
|Multiple threads want READ lock, no writer active|All granted concurrently|
|Thread wants WRITE lock, readers currently active|Must wait for all readers to finish|
|Thread wants READ lock, writer currently active|Must wait for writer to finish|
|Downgrade (hold write, then acquire read, then release write)|Supported|
|Upgrade (hold read, then try to acquire write)|NOT supported — deadlocks|

---

## Keywords

ReadWriteLock, ReentrantReadWriteLock, Read Lock, Write Lock, Lock Downgrading, Read-Heavy Workload

---

## Related Notes

[[32 - ReentrantLock]] [[36 - Deadlock]] [[52 - ConcurrentHashMap]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Implement a simple shared cache protected by `ReentrantReadWriteLock`, with `get()` using the read lock and `put()` using the write lock.
2. Confirm multiple reader threads can run concurrently by observing overlapping timestamps in their output.

**Medium:** 3. Reproduce this note's reader/writer timing demo, confirming the writer waits for all readers to finish. 4. Implement lock downgrading: acquire the write lock, make an update, acquire the read lock before releasing the write lock, then continue reading with the read lock held.

**Hard:** 5. Benchmark a read-heavy workload (e.g., 20 reader threads, 1 writer thread) using both a plain `ReentrantLock` and a `ReentrantReadWriteLock`, and measure the throughput difference on your machine.

---

## Mini Project

**Project: Read-Heavy Configuration Store**

Build a small console application that:

1. Implements a `ConfigStore` class protected by `ReentrantReadWriteLock`, holding a `Map<String, String>` of configuration values.
2. Simulates 20 "request handler" threads repeatedly reading configuration values, and 1 "admin" thread occasionally updating the configuration (e.g., every 2 seconds).
3. Measures total read throughput (reads/second) using `ReadWriteLock`, then re-runs the same simulation using a plain `ReentrantLock` for all access, comparing throughput.
4. Prints a summary demonstrating the performance benefit of `ReadWriteLock` for this specific read-heavy access pattern.

**Goal:** Build a realistic, measured demonstration of exactly when and why `ReadWriteLock` earns its added complexity.

---

## Common Interview Trap

**Trap Question:** _"If a thread already holds the read lock and needs to make an update, can it simply call `writeLock.lock()` to upgrade, do the update, then continue?"_

The trap is assuming lock upgrading works symmetrically to lock downgrading. The correct answer: **no** — attempting to acquire the write lock while still holding the read lock causes a **deadlock**. The write lock acquisition will wait for all current readers (including this very thread) to release their read locks first — but since this thread is now blocked waiting for the write lock, it will never reach the code that would release its own read lock, resulting in the thread waiting on itself indefinitely. The correct approach is to **release the read lock first**, then acquire the write lock separately (accepting that another thread could modify the data in the gap between these two steps, if that matters for your specific logic) — or restructure the logic to avoid needing this upgrade pattern altogether.

---

## Frequently Asked Questions

**Q: Does `ReentrantReadWriteLock` guarantee that writers won't be starved by a constant stream of readers?** A: Not by default in all implementations/modes — this depends on the specific fairness policy; the fair-mode constructor option is specifically designed to help prevent writer starvation, at some throughput cost (see [[38 - Starvation]] for the general concept).

**Q: Can I use `ReadWriteLock` with `Condition` objects for wait/notify-style coordination?** A: Yes — both the read lock and write lock support obtaining `Condition` objects via `newCondition()`, though this is a less common, more advanced pattern (see [[35 - Condition]] for the general `Condition` mechanism).

**Q: Is `ReentrantReadWriteLock` the only implementation of `ReadWriteLock` in the JDK?** A: It's the primary, most commonly used one; `StampedLock` (see [[34 - StampedLock]]) offers a newer, often higher-performance alternative with additional capabilities, at the cost of a more complex API.

**Q: Does the read lock support `tryLock()` and timed acquisition, like `ReentrantLock` does?** A: Yes — both the read lock and write lock objects returned by `ReentrantReadWriteLock` implement the full `Lock` interface, including `tryLock()` and `tryLock(timeout, unit)`.

---

## Revision Checklist

- [ ] I understand the core rule: multiple readers OR one exclusive writer, never both.
- [ ] I understand why lock upgrading (read → write) deadlocks, while downgrading (write → read) is safe.
- [ ] I can identify when `ReadWriteLock` is worth its added complexity (read-heavy workloads) versus when it isn't.
- [ ] I know both locks require the standard `try/finally` discipline.
- [ ] I understand the performance trade-off compared to a plain `ReentrantLock`.

---

## Flashcards

**Q:** Can multiple threads hold the read lock simultaneously? **A:** Yes.

**Q:** Can a reader and a writer hold their locks at the same time? **A:** No — a writer requires fully exclusive access, blocking all readers and other writers.

**Q:** Is upgrading from read lock to write lock supported? **A:** No — it causes a deadlock. Only downgrading (write → read) is safely supported.

**Q:** When is `ReadWriteLock` worth its added complexity? **A:** When reads significantly outnumber writes.

**Q:** What is lock downgrading? **A:** Acquiring the read lock while still holding the write lock, then releasing the write lock — ending up with just the read lock, with no gap for another writer to interfere.

---

## References

- Oracle Java Tutorials — Lock Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html
- `java.util.concurrent.locks.ReadWriteLock` API Documentation
- `java.util.concurrent.locks.ReentrantReadWriteLock` API Documentation