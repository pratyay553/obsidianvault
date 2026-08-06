# 64 - Memory Consistency

Tags: #java-memory-model #section11
Related: [[62 - Java Memory Model]] | [[63 - Happens Before]] | [[29 - Volatile]] | [[65 - False Sharing]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand memory consistency errors — what they are, why they happen, and how to prevent them.

---

## Beginner Explanation
A **memory consistency error** occurs when different threads have inconsistent views of the same data. Thread A writes a value, but Thread B reads a stale (old) version because the write hasn't been flushed from A's CPU cache to shared memory yet.

---

## Analogy
Two editors working on the same document offline. Editor A saves "Chapter 3 is done." Editor B still sees an old copy without that update. They have inconsistent views of the document. Without a sync mechanism, B's edits can conflict with A's.

---

## Technical Deep Dive

### Why inconsistency happens

Modern CPUs have multi-level caches (L1, L2, L3). Each core has its own L1/L2. A write by Core A goes to Core A's cache first and may not reach main memory (or other cores' caches) for a long time — until a cache flush occurs.

```
Core A (Thread A)          Core B (Thread B)
  L1 cache: x=1              L1 cache: x=0 (stale)
       ↕                          ↕
  L2 cache                  L2 cache
       ↕                          ↕
         Main Memory: x=0 (not yet updated)
```

### Types of memory consistency errors

**1. Visibility error** — read sees stale value
```java
boolean flag = false;
// Thread A: flag = true;
// Thread B: while (!flag) {} // may loop forever
```

**2. Reordering error** — actions appear in different order to different threads
```java
// Thread A:
x = 1;
published = true; // compiler/CPU may reorder these!
// Thread B:
if (published) doWith(x); // may see published=true but x=0
```

**3. Atomicity error** — partial writes visible
```java
long value = 0;
// Thread A: value = Long.MAX_VALUE; // may be two 32-bit writes
// Thread B: long v = value; // may see a half-written value
```

### Hardware memory models vs JMM
The JMM is more relaxed than strict sequential consistency (SC) but defines a safe programming model. Hardware models (x86, ARM) have their own rules:
- **x86 (TSO)**: relatively strong — stores to same address are ordered; load-store reordering limited
- **ARM/POWER**: very weak — almost everything can be reordered without barriers

The JVM inserts memory barriers (`mfence`, `dmb`) as needed to enforce JMM guarantees on the running hardware.

### How to achieve memory consistency
1. **`volatile`** — prevents visibility errors + prevents reordering around the volatile access
2. **`synchronized`** — full memory barrier on lock/unlock
3. **`java.util.concurrent` utilities** — all use proper barriers internally
4. **`AtomicXxx`** — atomic operations with volatile semantics

---

## Code

```java
public class MemoryConsistencyDemo {
    // BAD — potential visibility + reordering
    static int data = 0;
    static boolean flag = false;

    // GOOD — volatile ensures both visibility and ordering
    static volatile boolean flagV = false;

    public static void main(String[] args) throws InterruptedException {
        // Demonstrate the fix
        Thread writer = new Thread(() -> {
            data = 99;
            flagV = true; // volatile write — HB fence, also prevents reorder with data=99
        });
        Thread reader = new Thread(() -> {
            while (!flagV) {} // volatile read
            // Guaranteed: flagV=true HB this point, and data=99 HB flagV=true
            // So data=99 is visible here
            System.out.println("Read data: " + data); // always 99
        });
        writer.start(); reader.start();
        writer.join(); reader.join();
    }
}
```

---

## Interview Questions

**Q: What is a memory consistency error?**
> When two threads have inconsistent (different) views of the same shared variable because writes haven't propagated through CPU caches to all cores.

**Q: Can reordering cause bugs even on a single CPU?**
> On a single CPU, the hardware preserves program-order semantics for that CPU. But the compiler itself can reorder operations, and other threads on other CPUs won't see a guaranteed order without memory barriers.

**Q: What is the difference between `volatile` and `synchronized` for memory consistency?**
> Both establish happens-before. `volatile` prevents reordering around a single variable access and ensures visibility. `synchronized` creates a full mutual exclusion section with happens-before on lock/unlock — stronger but coarser.

---

## Cheat Sheet
| Error Type | Cause | Fix |
|---|---|---|
| Visibility | CPU cache not flushed | `volatile`, `synchronized` |
| Reordering | CPU/compiler optimization | Memory barriers (`volatile`, `synchronized`) |
| Atomicity | Split 64-bit writes | `volatile long/double`, `AtomicLong` |
| Enforcement | JMM inserts hw barriers | Happens-before relation |
