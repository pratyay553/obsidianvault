# Volatile

## Objective

By the end of this note, you will understand:

- Exactly what the `volatile` keyword guarantees, precisely
- What `volatile` does NOT guarantee (a very common point of confusion)
- When `volatile` is the right tool versus when you need `synchronized` or atomic classes
- How `volatile` relates to the happens-before relationship

This note builds directly on [[28 - Visibility Problem]] — `volatile` is the primary, lightweight tool Java provides specifically to solve that problem for individual variables.

---

## Why should I learn this?

`volatile` is one of the most **misunderstood** keywords in Java. Many developers either avoid it entirely (not realizing it's the correct, idiomatic fix for certain problems) or misuse it (assuming it provides broader guarantees than it actually does, like atomicity for compound operations). Understanding its precise, limited guarantee — visibility and ordering, but **not** atomicity — is essential for using it correctly and for recognizing when it's insufficient.

---

## Beginner Explanation

Recall from [[28 - Visibility Problem]]: normally, a thread might cache a variable's value locally and never notice when another thread updates it. The `volatile` keyword is a simple instruction you attach to a field declaration that tells the JVM: **"never let any thread cache this variable locally — always read the true, current value directly from main memory, and always write changes immediately and visibly back to main memory."**

Think of it like designating a specific whiteboard in an office as "always check this board directly, never rely on your own notes about it" — everyone agrees to always look at the actual, current whiteboard content rather than trusting a personal, potentially-outdated copy.

---

## Real World Analogy

**Airport Departure Board Analogy**

Imagine an airport's main departure board, showing live flight statuses. Passengers don't write down the gate number once and trust their own note for the rest of the day — they know gate assignments can change, so they make a habit of glancing back at the **actual, live board** whenever they need current information, rather than relying on a personal note that might now be stale.

A `volatile` field is exactly like this "always check the live board" agreement — every thread that reads a `volatile` variable is guaranteed to see the most recent value written by any other thread, rather than a potentially stale, locally cached copy.

---

## Technical Explanation

### Declaring a Volatile Field

```java
public class StatusFlag {
    private volatile boolean running = true;
}
```

`volatile` can only be applied to **fields** (instance or static) — it cannot be applied to local variables, method parameters, or method return types.

### What `volatile` Guarantees

1. **Visibility**: Any write to a `volatile` variable by one thread is guaranteed to be visible to any other thread that subsequently reads that same variable.
2. **Ordering (no reordering across the volatile access)**: The JVM guarantees that operations occurring **before** a `volatile` write, in program order, cannot be reordered to happen **after** that write from the perspective of another thread observing it — and similarly for reads. This connects to the happens-before relationship (see [[63 - Happens Before]]).
3. **No caching of the value in CPU registers/local thread caches** — every read goes to (an appropriately synchronized view of) main memory, and every write is immediately flushed there.

### What `volatile` Does NOT Guarantee

This is the most commonly tested, most commonly misunderstood aspect:

```java
private volatile int count = 0;

public void increment() {
    count++; // STILL NOT ATOMIC, even though count is volatile!
}
```

`volatile` does **not** provide atomicity for compound (multi-step) operations. `count++` is still fundamentally a read-modify-write sequence (see [[22 - Synchronization]], [[26 - Race Condition]]) — `volatile` only guarantees that each individual read and each individual write is properly visible; it does **not** make the combination of "read, then modify, then write" atomic as a whole. Two threads can still interleave their reads and writes of a `volatile int` and produce a lost update, exactly like a non-volatile `int`.

**For atomic compound operations, you need either `synchronized` or, more idiomatically for simple counters, `AtomicInteger`** (see [[30 - Atomic Variables]]).

### When `volatile` IS Sufficient

`volatile` is the correct, sufficient tool specifically when:

- A single thread writes, and one or more other threads only read (no compound read-modify-write operations by multiple threads).
- You need a simple status flag, configuration value, or "latest snapshot" reference that's updated as a whole (a single, simple assignment), not incrementally modified.

```java
private volatile boolean shutdownRequested = false;

// Writer thread:
public void requestShutdown() {
    shutdownRequested = true; // simple, whole-value write
}

// Reader thread:
public void runLoop() {
    while (!shutdownRequested) {
        // do work
    }
}
```

This is a textbook-correct use of `volatile` — exactly the fix demonstrated in [[28 - Visibility Problem]].

---

## Internal Working

At a conceptual JVM level, when a field is declared `volatile`:

1. Every **write** to that field is treated as if it includes a "flush to main memory" step, making the new value immediately visible to other threads, and is also treated as a synchronization point preventing certain instruction reorderings around it.
2. Every **read** of that field is treated as if it includes a "refresh from main memory" step, ensuring the thread never uses a stale, locally cached value.
3. The JVM prevents the compiler/JIT from applying the specific class of optimization described in [[28 - Visibility Problem]] (caching the value in a register and skipping re-reads) for this particular field.
4. Unlike `synchronized`, **no lock is acquired** — there's no mutual exclusion, no blocking, and no risk of contention/deadlock purely from `volatile` access; it's a much lighter-weight mechanism, but with correspondingly weaker guarantees (no atomicity for compound operations).

---

## Memory Diagram

```
        Main Memory
   -----------------------
   |  volatile flag: false |  <- always the authoritative, current value
   -----------------------
        ^              ^
        |              |
   Thread A          Thread B
   ALWAYS reads      ALWAYS writes
   current value      -> immediately
   from main memory    visible to
   (no stale cache)    all readers
```

---

## Execution Flow

```
Thread A: while (!volatileFlag) { ... }
        |
        v
Each loop iteration: read volatileFlag DIRECTLY
from main memory (no caching allowed)
        |
   (meanwhile, concurrently)
        |
        v
Thread B: volatileFlag = true;
        |
        v
Write immediately, visibly flushed to main memory
        |
        v
Thread A's NEXT read reliably sees the updated value
        |
        v
Loop exits promptly, correctly
```

---

## Code Example

```java
import java.util.concurrent.atomic.AtomicInteger;

public class VolatileDemo {

    // Correct use of volatile: simple flag, single writer, visibility-only need
    static volatile boolean stopRequested = false;

    // INCORRECT use of volatile: compound operation still needs atomicity
    static volatile int volatileCounter = 0;

    public static void main(String[] args) throws InterruptedException {

        // --- Demo 1: volatile correctly fixes a stop-flag visibility issue ---
        Thread worker = new Thread(() -> {
            long iterations = 0;
            while (!stopRequested) {
                iterations++;
            }
            System.out.println("Worker stopped after " + iterations + " iterations (volatile works correctly here).");
        });
        worker.start();
        Thread.sleep(500);
        stopRequested = true;
        worker.join();

        // --- Demo 2: volatile does NOT fix a compound read-modify-write race ---
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 100_000; j++) {
                    volatileCounter++; // STILL a race condition, despite volatile!
                }
            });
        }
        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();

        System.out.println("volatileCounter result: " + volatileCounter +
                " (expected 1000000 -- likely LESS, proving volatile alone doesn't fix this)");
    }
}
```

### Line-by-line explanation

1. **Demo 1** shows `volatile` correctly solving a pure visibility problem — a simple flag, one writer, prompt and reliable propagation to the reading thread.
2. **Demo 2** shows `volatile`'s **limitation** — even though `volatileCounter` is declared `volatile`, the `count++` pattern across 10 threads still produces **lost updates**, because `volatile` doesn't make the read-modify-write sequence atomic. The final result will very likely be **less than** 1,000,000, exactly like the non-volatile version from [[22 - Synchronization]]'s example.
3. This side-by-side comparison is the clearest possible demonstration of `volatile`'s precise, limited guarantee.

---

## Dry Run

**Demo 1 output (reliable, every run):**

```
Worker stopped after 48213840 iterations (volatile works correctly here).
```

**Demo 2 output (varies by run, but consistently shows a shortfall):**

```
volatileCounter result: 941823 (expected 1000000 -- likely LESS, proving volatile alone doesn't fix this)
```

---

## Common Mistakes

1. **Assuming `volatile` makes compound operations (`count++`, `list.add()`, etc.) atomic.**
    
    - As demonstrated, it does not — `volatile` only guarantees visibility and ordering for individual reads/writes, not atomicity across multiple operations.
2. **Using `volatile` as a blanket substitute for `synchronized` in all cases.**
    
    - `volatile` is appropriate for simple flags/single-writer scenarios; for anything requiring atomic compound operations or coordinated multi-variable updates, `synchronized` or explicit locks/atomic classes are needed instead.
3. **Applying `volatile` to a field that's already properly protected by consistent `synchronized` access, assuming it adds extra safety.**
    
    - This is usually unnecessary and can be a sign of confused reasoning about which mechanism is actually providing the guarantee — pick one consistent mechanism per piece of shared state.
4. **Not realizing `volatile` cannot be applied to local variables.**
    
    - `volatile` is only valid on field declarations (instance or static); it has no meaning for local variables in Java (each thread already has its own private stack/local variables, so there's no cross-thread visibility concern for genuinely local variables).

---

## Production Usage

- **Shutdown/stop flags** for background threads and services — the single most common, textbook-correct use of `volatile` in real production code.
- **Configuration "latest snapshot" references**: e.g., a `volatile` reference to an immutable configuration object, atomically swapped out as a whole when configuration changes, allowing readers to always see either the old or new complete configuration, never a partially-updated one.
- **Double-checked locking** (a historical singleton initialization pattern) relies on `volatile` specifically to work correctly in modern Java — a well-known example worth researching for deeper understanding.
- **Feature flags/circuit breaker states** in resilience libraries often use `volatile` fields for their simple, frequently-read, infrequently-written boolean or enum status values.

---

## Interview Questions

**Beginner:**

1. What does the `volatile` keyword guarantee?
2. Can `volatile` be applied to local variables?
3. Does `volatile` prevent race conditions on compound operations like `count++`?

**Intermediate:** 4. Give an example of a correct use case for `volatile`, and an example of a case where `volatile` alone is insufficient. 5. How does `volatile` relate to the happens-before relationship? 6. What's the performance difference between `volatile` and `synchronized`?

**Advanced:** 7. Why doesn't `volatile` provide atomicity, even though it ensures every individual read/write is properly visible? 8. Explain how `volatile` prevents the specific JIT compiler optimization responsible for the visibility problem described in [[28 - Visibility Problem]].

**Production-Level:** 9. Describe a real production scenario where `volatile` is the correct, sufficient tool, and one where it would be insufficient and `synchronized`/atomic classes are needed instead. 10. How would you recognize, during code review, that a `volatile` field is being misused for something requiring true atomicity?

---

## Follow-up Questions

- Is `volatile` ever used together with `synchronized` on the same field? _(This is uncommon and often redundant/confusing — if a field is consistently accessed only within `synchronized` blocks using a single, consistent lock, that already provides the necessary visibility guarantee; adding `volatile` on top is usually unnecessary, though not incorrect.)_
- Does `volatile` add significant performance overhead? _(Generally modest — it disables certain CPU/compiler optimizations and may involve memory barrier instructions, but it's substantially lighter-weight than acquiring a lock, since there's no blocking/contention involved.)_
- Can `volatile` be applied to object references, not just primitives? _(Yes — `volatile` works on any field type, including object references; when applied to a reference, it guarantees visibility of the reference itself being updated, but does NOT automatically make the referenced object's own internal fields thread-safe — that's a separate concern.)_

---

## Best Practices

- Use `volatile` specifically for simple, single-writer (or safely-published) flags and status fields where only visibility (not atomicity) is needed.
- Never use `volatile` alone for counters, accumulators, or any compound read-modify-write logic — use `synchronized` or atomic classes (`AtomicInteger`, etc.) instead (see [[30 - Atomic Variables]]).
- Be clear, in your own mental model and in code comments if helpful, about exactly what guarantee a given `volatile` field is providing — this avoids the common mistake of over-trusting its scope.
- Avoid redundantly combining `volatile` with consistent `synchronized` access to the same field — pick one mechanism.

---

## Performance Notes

- `volatile` access is generally cheaper than acquiring a lock (`synchronized`), since there's no blocking or contention involved — just a memory-visibility guarantee.
- However, `volatile` reads/writes are still somewhat more expensive than plain field access, due to the memory barrier instructions and disabled optimizations involved — don't apply it indiscriminately to fields that don't actually need cross-thread visibility.

---

## Comparison Table

|Aspect|`volatile`|`synchronized`|`AtomicInteger` (see [[30 - Atomic Variables]])|
|---|---|---|---|
|Guarantees visibility?|Yes|Yes|Yes|
|Guarantees atomicity for compound ops?|No|Yes|Yes (for its specific supported operations)|
|Can cause blocking/contention?|No|Yes|No (typically uses CAS, see [[31 - CAS]])|
|Best for|Simple flags, single-writer scenarios|General-purpose critical sections|Simple atomic counters/accumulators|

---

## Summary

- `volatile` guarantees visibility (and ordering) for a single field — writes by one thread are reliably, promptly visible to other threads reading that field.
- It does **not** guarantee atomicity for compound operations like `count++` — this remains a race condition even on a `volatile` variable.
- It's the correct, lightweight fix specifically for simple, single-writer flags/status fields (like shutdown signals).
- For anything requiring atomic compound operations, use `synchronized` or purpose-built atomic classes instead.
- `volatile` is cheaper than `synchronized` (no blocking/locking), but this efficiency comes with a correspondingly narrower guarantee.

---

## Cheat Sheet

|Question|Answer|
|---|---|
|Does `volatile` fix visibility?|Yes|
|Does `volatile` fix atomicity for `count++`?|No|
|Can `volatile` be used on local variables?|No — fields only|
|Is `volatile` cheaper than `synchronized`?|Yes — no blocking/locking involved|
|Correct use case|Simple flags, single-writer status fields|
|Wrong use case|Counters, accumulators, compound operations|

---

## Keywords

volatile, Visibility, Ordering, Happens-Before, Memory Barrier, Atomicity, Stop Flag, AtomicInteger

---

## Related Notes

[[28 - Visibility Problem]] [[22 - Synchronization]] [[26 - Race Condition]] [[30 - Atomic Variables]] [[63 - Happens Before]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's Demo 1 (correct `volatile` stop flag usage) and confirm it reliably stops promptly.
2. Reproduce Demo 2 (volatile counter race) and confirm the result is less than expected, at least on some runs.

**Medium:** 3. Fix Demo 2's `volatileCounter` race using `synchronized` instead of relying on `volatile` alone, and confirm the result becomes reliably correct. 4. Fix Demo 2's `volatileCounter` race using `AtomicInteger` instead, and compare the code simplicity/performance to the `synchronized` version.

**Hard:** 5. Research and implement the classic "double-checked locking" singleton pattern, first WITHOUT `volatile` (understanding why it's historically broken), then WITH `volatile` (understanding exactly how it fixes the issue).

---

## Mini Project

**Project: Volatile Correctness Explorer**

Build a small console application that:

1. Implements three versions of a shared counter: plain (no `volatile`, no `synchronized`), `volatile`-only, and properly `synchronized`.
2. Runs identical concurrent increment workloads against all three.
3. Reports the final result for each, run multiple times, clearly showing that plain and `volatile`-only versions both frequently produce incorrect (lower than expected) results, while the `synchronized` version is always correct.
4. Includes a brief written summary (in code comments or console output) explaining exactly why `volatile` didn't fix the counter issue, tying back to this note's core lesson.

**Goal:** Build definitive, empirical, hands-on proof of `volatile`'s precise and limited guarantee — cementing the single most commonly misunderstood aspect of this keyword.

---

## Common Interview Trap

**Trap Question:** _"If I make a shared counter `volatile`, is it now safe for multiple threads to increment it concurrently without any other synchronization?"_

The trap is assuming `volatile` provides a general "thread safety" guarantee. The correct answer: **no** — `volatile` only guarantees that each individual read and write of the variable is visible and properly ordered across threads; it does **not** make compound operations like `count++` atomic. Since `count++` is actually a read-then-modify-then-write sequence, two threads can still interleave these steps and produce lost updates, exactly as if the variable weren't `volatile` at all. For a genuinely thread-safe, concurrently-incrementable counter, you need either a `synchronized` block/method wrapping the entire increment operation, or (more idiomatically) `AtomicInteger.incrementAndGet()` (see [[30 - Atomic Variables]]).

---

## Frequently Asked Questions

**Q: Is `volatile` the same as declaring a field `final`?** A: No — they solve entirely different problems. `final` means the field's value (or reference) cannot be reassigned after initial construction; `volatile` means visibility/ordering guarantees apply to a field that CAN change over time. A field can be both, neither, or either independently, depending on your needs.

**Q: Does marking a reference field `volatile` make the object it points to thread-safe?** A: No — it only guarantees visibility of the **reference itself** changing (i.e., other threads will promptly see when the field starts pointing to a different object); it says nothing about the thread-safety of the referenced object's own internal state/fields.

**Q: Is `volatile` commonly used in modern, high-level production code, or mostly in low-level libraries?** A: Both — while lower-level concurrent library/framework code uses it extensively (often as a building block for more complex synchronization), it's also genuinely common and appropriate in everyday application code for simple flags (like shutdown signals).

**Q: Does `volatile` guarantee ordering only for the volatile variable itself, or can it affect other, non-volatile variables too?** A: Interestingly, it can provide ordering guarantees for surrounding non-volatile variables too, due to the happens-before relationship it establishes — see [[63 - Happens Before]] for the full detail on this more advanced aspect.

---

## Revision Checklist

- [ ] I can precisely state what `volatile` guarantees (visibility and ordering) and what it does NOT guarantee (atomicity for compound operations).
- [ ] I can identify correct use cases for `volatile` (simple flags) versus cases requiring `synchronized`/atomic classes.
- [ ] I understand why `count++` on a `volatile int` is still a race condition.
- [ ] I know `volatile` can only be applied to fields, not local variables.
- [ ] I understand the performance trade-off between `volatile` and `synchronized`.

---

## Flashcards

**Q:** What does `volatile` guarantee? **A:** Visibility and ordering for reads/writes of that specific field across threads.

**Q:** Does `volatile` make `count++` atomic? **A:** No — it's still a race condition, since `count++` is a compound read-modify-write operation.

**Q:** Can `volatile` be applied to local variables? **A:** No — only to fields (instance or static).

**Q:** What's a textbook-correct use case for `volatile`? **A:** A simple shutdown/stop flag, written by one thread and read by another.

**Q:** Is `volatile` more or less expensive than `synchronized`? **A:** Generally less expensive — no blocking/locking is involved, just visibility guarantees.

---

## References

- Oracle Java Tutorials — Atomic Access: https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
- JLS (Java Language Specification) — Chapter 17: Threads and Locks (`volatile` semantics)
- `java.util.concurrent.atomic` package documentation