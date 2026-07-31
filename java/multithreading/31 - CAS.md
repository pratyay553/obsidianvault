# CAS

## Objective

By the end of this note, you will understand:

- What Compare-And-Swap (CAS) is, at the hardware/CPU level
- Exactly how atomic classes (from [[30 - Atomic Variables]]) use CAS internally
- The ABA problem — a subtle, classic CAS-related pitfall
- Why CAS-based (lock-free) algorithms are foundational to high-performance concurrent Java

This note closes out **Section 4: Synchronization Fundamentals**, going one level deeper than [[30 - Atomic Variables]] into the actual hardware mechanism that makes lock-free atomics possible.

---

## Why should I learn this?

Understanding CAS transforms atomic classes from "a magic API that just works" into "a specific, well-understood technique with real trade-offs." This is exactly the kind of depth that distinguishes a candidate who has genuinely studied concurrency internals from one who has only memorized API method names. CAS is also the foundational building block behind lock-free data structures throughout `java.util.concurrent` — understanding it deepens your grasp of `ConcurrentHashMap`, `ConcurrentLinkedQueue`, and more.

---

## Beginner Explanation

**Compare-And-Swap (CAS)** is a special CPU instruction that does three things **as a single, indivisible hardware operation**:

1. Compare a memory location's current value to an **expected** value.
2. If they match, **swap in** a new value.
3. Report whether the swap succeeded or failed.

The critical property is that this entire "compare, then swap" sequence happens **atomically at the hardware level** — no other CPU core can possibly interleave in the middle of it, because the CPU itself guarantees this as a single, uninterruptible instruction.

Think of it like a vending machine slot that only accepts your coin if the slot is **currently empty**: you try to insert your coin, and the machine's mechanism atomically checks "is this slot empty right now?" and, if so, accepts your coin in one single, physical motion — there's no possible moment where two people's coins could both slip in at exactly the same instant, because the physical mechanism itself prevents it.

---

## Real World Analogy

**Bank Analogy**

Imagine a bank vault with a special lock that works like this: you present a note saying "I believe the current balance is $1000; please update it to $900 if that's correct." The vault's mechanism **atomically** checks the current balance against your stated expectation and, if it matches, performs the update in one indivisible motion. If someone else had already changed the balance to $950 in between when you checked and when you submitted your request, the vault's mechanism detects the mismatch and **rejects** your update entirely — telling you "the balance wasn't what you thought; try again with the current information."

This is exactly the CAS pattern: you never blindly overwrite a value — you always specify what you **expect** the current value to be, and the operation only succeeds if reality matches your expectation, entirely as one atomic step.

---

## Technical Explanation

### The CAS Operation, Precisely

CAS takes three arguments:

1. A memory location (address).
2. An **expected** current value.
3. A **new** value to set.

```
CAS(memoryLocation, expectedValue, newValue):
    atomically:
        if memoryLocation.currentValue == expectedValue:
            memoryLocation.currentValue = newValue
            return true  (success)
        else:
            return false (failure -- value was NOT what was expected)
```

This entire operation is implemented as a **single CPU instruction** on modern hardware (e.g., `CMPXCHG` on x86 architectures), meaning there is no possible window for another thread/core to interleave in the middle of the compare-and-set — the hardware itself provides this atomicity guarantee, without needing an OS-level lock at all.

### How Java's Atomic Classes Use CAS

From [[30 - Atomic Variables]], `AtomicInteger.incrementAndGet()` works internally, roughly, like this:

```java
public final int incrementAndGet() {
    int current;
    int next;
    do {
        current = get();          // read current value
        next = current + 1;       // compute desired new value
    } while (!compareAndSet(current, next)); // retry until CAS succeeds
    return next;
}
```

This **compare-and-retry loop** is the defining pattern of lock-free programming: instead of blocking other threads out with a lock while you make your change, you **optimistically** attempt your change, and if another thread beat you to it, you simply **recompute and try again** — no thread is ever forced to sleep/block waiting for another.

### The ABA Problem — A Classic CAS Pitfall

CAS checks whether a value is **still equal** to what you expect — but it cannot detect if the value changed **and then changed back** to the same value in between your read and your CAS attempt. This is called the **ABA problem**:

1. Thread A reads a shared reference, currently pointing to Object `X`.
2. Thread A is paused (context switch).
3. Thread B changes the reference from `X` to `Y`, and then **back** to `X` again (perhaps as part of some other legitimate, unrelated operation, like removing and re-adding a node in a lock-free stack).
4. Thread A resumes, performs its CAS check: "is the reference still `X`?" — **Yes**, it is (again) — so the CAS succeeds.
5. **But** Thread A's CAS succeeding might be **incorrect** in context, because the object at that reference isn't really "the same X" in a meaningful sense anymore — something happened in between that Thread A's simple value-equality check couldn't detect (e.g., in a lock-free stack, the _identity_ of nodes further down the stack might have changed even though the top pointer coincidentally matches again).

**The standard fix**: use a **versioned reference** — pairing the actual value with a monotonically increasing version/stamp number, so that even if the value returns to its original state, the version number will have changed, and the CAS will correctly detect that "something happened in between," even though the raw value looks unchanged. Java's `AtomicStampedReference` class exists specifically to address this.

```java
import java.util.concurrent.atomic.AtomicStampedReference;

AtomicStampedReference<String> ref = new AtomicStampedReference<>("X", 0);

int[] stampHolder = new int[1];
String current = ref.get(stampHolder); // gets value AND current stamp

// compareAndSet checks BOTH the value AND the stamp
boolean success = ref.compareAndSet(current, "Y", stampHolder[0], stampHolder[0] + 1);
```

---

## Internal Working

Step-by-step of a CAS-based retry loop under contention (two threads incrementing concurrently):

1. Thread A reads current value: `5`.
2. Thread B reads current value: `5` (before A has updated anything).
3. Thread A computes `6`, executes CAS(expected=5, new=6) — **succeeds** (value was still 5). Value is now `6`.
4. Thread B computes `6` (based on its earlier read of 5), executes CAS(expected=5, new=6) — **fails** (value is now 6, not 5 anymore).
5. Thread B **retries**: re-reads current value, now `6`. Computes `7`. Executes CAS(expected=6, new=7) — **succeeds**. Value is now `7`.
6. Final result: `7` — correct, both increments counted, with zero blocking involved anywhere in this process.

---

## Memory Diagram

```
        Shared memory location
   -----------------------------
   |  value: 5                  |
   -----------------------------
        ^              ^
        |              |
   Thread A         Thread B
   CAS(5, 6)        CAS(5, 6)
   -> SUCCESS       -> attempted at
   value = 6           nearly same time,
                        but value already
                        changed to 6
                        -> FAILS, must retry
                        CAS(6, 7) -> SUCCESS
                        value = 7
```

---

## Execution Flow

```
Thread wants to atomically update a value
        |
        v
Read current value (optimistic read, no lock)
        |
        v
Compute desired new value based on that read
        |
        v
Attempt CAS(expected=old_read, new=computed)
        |
   -----------------------------
   |                           |
 SUCCESS                    FAILURE
 (value hadn't changed)    (another thread changed it first)
   |                           |
   v                           v
Done, return new value    RETRY: loop back to "Read current value"
                            (no blocking -- just try again)
```

---

## Code Example

```java
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class CasDemo {

    public static void main(String[] args) {

        // --- Manually implementing an atomic increment using raw compareAndSet ---
        AtomicInteger value = new AtomicInteger(0);

        Runnable manualIncrementLoop = () -> {
            for (int i = 0; i < 50_000; i++) {
                int current, next;
                int retries = 0;
                do {
                    current = value.get();
                    next = current + 1;
                    retries++;
                } while (!value.compareAndSet(current, next)); // manual CAS retry loop
            }
        };

        Thread t1 = new Thread(manualIncrementLoop);
        Thread t2 = new Thread(manualIncrementLoop);
        try {
            t1.start(); t2.start();
            t1.join(); t2.join();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Manual CAS-based counter result: " + value.get() + " (expected 100000)");

        // --- Demonstrating AtomicStampedReference to avoid the ABA problem ---
        AtomicStampedReference<String> stampedRef = new AtomicStampedReference<>("X", 0);

        int[] stampHolder = new int[1];
        String currentVal = stampedRef.get(stampHolder);
        System.out.println("Initial value: " + currentVal + ", stamp: " + stampHolder[0]);

        // Simulate "ABA": someone changes X -> Y -> X again, bumping the stamp each time
        stampedRef.compareAndSet("X", "Y", 0, 1);
        stampedRef.compareAndSet("Y", "X", 1, 2);

        // Our ORIGINAL stamp (0) is now stale, even though the VALUE looks the same ("X")
        boolean success = stampedRef.compareAndSet(currentVal, "Z", stampHolder[0], stampHolder[0] + 1);
        System.out.println("CAS with stale stamp succeeded? " + success + " (should be FALSE -- ABA correctly detected)");
    }
}
```

### Line-by-line explanation

1. The manual `compareAndSet()` retry loop reimplements exactly what `incrementAndGet()` does internally — this makes the CAS mechanism fully explicit and visible rather than hidden inside a convenience method.
2. The result reliably equals `100000` (50,000 × 2 threads), demonstrating correctness despite zero locking.
3. The `AtomicStampedReference` section demonstrates the ABA problem's fix: even though the value returns to `"X"`, the **stamp** (version number) has changed from `0` to `2`, so a CAS attempt using the original stale stamp (`0`) correctly **fails**, protecting against the ABA scenario.

---

## Dry Run

**Manual CAS counter:**

```
Manual CAS-based counter result: 100000 (expected 100000)
```

**ABA demonstration:**

```
Initial value: X, stamp: 0
CAS with stale stamp succeeded? false (should be FALSE -- ABA correctly detected)
```

---

## Common Mistakes

1. **Assuming CAS-based retry loops can loop "forever" in practice under normal conditions.**
    
    - While theoretically possible under pathological, extremely high contention, in practice CAS retry loops converge quickly for the vast majority of real workloads — this isn't a practical everyday concern for typical applications.
2. **Not considering the ABA problem when implementing custom lock-free data structures.**
    
    - If you're building custom CAS-based logic involving object references (not just simple counters), the ABA problem is a genuine, subtle risk worth specifically considering — using `AtomicStampedReference` or `AtomicMarkableReference` when appropriate.
3. **Believing CAS eliminates all synchronization overhead entirely.**
    
    - CAS still involves real CPU-level cost (memory barriers, potential cache-line contention across cores) — it's cheaper than lock-based blocking in many cases, but it's not literally free.
4. **Confusing CAS (a hardware-level mechanism) with `compareAndSet()` (the Java method that exposes this mechanism).**
    
    - The Java method is a thin wrapper around the underlying hardware CAS instruction (via `sun.misc.Unsafe` historically, or `VarHandle` in modern Java) — understanding this connection helps clarify why it's so efficient.

---

## Production Usage

- **Every atomic class** (`AtomicInteger`, `AtomicLong`, `AtomicReference`, etc.) in `java.util.concurrent.atomic` is built directly on CAS.
- **`ConcurrentHashMap`'s internal bucket/node updates** rely heavily on CAS operations for much of their lock-free (or minimally-locked) behavior, especially in modern JDK versions.
- **Lock-free queues and stacks** (like `ConcurrentLinkedQueue`) are classic examples of data structures implemented entirely using CAS-based algorithms, avoiding locks for high-throughput concurrent access.
- **`LongAdder`/`DoubleAdder`** (advanced high-throughput counters, an evolution beyond simple `AtomicLong` for extremely high-contention scenarios) use a more sophisticated CAS-based striping technique internally.

---

## Interview Questions

**Beginner:**

1. What does CAS stand for, and what does it do?
2. Is CAS a software or hardware-level mechanism?
3. What are the three "arguments" involved in a CAS operation?

**Intermediate:** 4. How do Java's atomic classes use CAS internally to implement operations like `incrementAndGet()`? 5. What happens when a CAS operation fails? 6. What is the ABA problem?

**Advanced:** 7. Why is CAS generally considered more efficient than lock-based synchronization under moderate contention? 8. How does `AtomicStampedReference` solve the ABA problem?

**Production-Level:** 9. Name a real JDK data structure that relies on CAS internally, and briefly explain how. 10. In what scenario might a CAS-based retry loop actually underperform compared to a well-designed lock-based approach?

---

## Follow-up Questions

- Does CAS require any operating system involvement, like a system call? _(No — CAS is a pure CPU instruction, executed entirely at the hardware level, with no OS involvement or context switch required, which is exactly why it's so much cheaper than lock-based blocking.)_
- Can CAS be used to atomically update MULTIPLE, unrelated memory locations at once? _(No — standard CAS only operates on a single memory location at a time; atomically coordinating multiple independent locations requires either a lock-based approach or more advanced/specialized techniques.)_
- Is the ABA problem a concern for simple `AtomicInteger` counters? _(Generally not in practice — since a counter's value doesn't have any meaningful "identity" beyond its numeric value, having a counter return to the same value it had before isn't actually a problem; ABA is primarily a concern for object references in lock-free data structures, where "same value" doesn't necessarily mean "nothing meaningful happened in between.")_

---

## Best Practices

- Trust and use the JDK's built-in atomic classes for standard use cases — they already correctly handle the CAS retry loop and relevant edge cases internally.
- If implementing custom CAS-based logic involving object references (not simple primitives), specifically consider whether the ABA problem could affect your design, and use `AtomicStampedReference`/`AtomicMarkableReference` if so.
- Understand CAS conceptually even if you rarely write raw CAS retry loops yourself — it deepens your understanding of why atomic classes and many `java.util.concurrent` internals behave the way they do.

---

## Performance Notes

- CAS is generally very fast — a single CPU instruction with no OS/context-switch involvement — making it the foundation of high-performance, low-contention concurrent code.
- Under **extremely** high contention (many threads constantly colliding), CAS retry loops can degrade in throughput due to repeated failed attempts and cache-line contention across CPU cores — this is precisely the scenario that led to the development of more advanced constructs like `LongAdder`, which use striping to reduce this specific contention pattern.

---

## Comparison Table

|Aspect|CAS (Lock-Free)|Lock-Based (`synchronized`)|
|---|---|---|
|Blocking?|No — retries instead|Yes|
|OS involvement|None (pure CPU instruction)|Yes (for contended locks)|
|Performance under low/moderate contention|Excellent|Good|
|Performance under extreme contention|Can degrade (many retries)|More predictable, but still costly|
|Complexity to implement custom logic correctly|Higher (must handle ABA, retry loops)|Lower (more straightforward mental model)|

---

## Summary

- CAS (Compare-And-Swap) is a hardware-level CPU instruction that atomically compares a value to an expected value and swaps in a new value only if they match.
- Java's atomic classes (`AtomicInteger`, etc.) are built directly on top of CAS, using an internal read-compute-CAS retry loop.
- CAS-based (lock-free) programming avoids blocking entirely — failed attempts simply retry, rather than causing a thread to wait.
- The **ABA problem** is a subtle risk when a value returns to its original state after intermediate changes, which simple CAS cannot detect — `AtomicStampedReference` fixes this using versioned stamps.
- CAS underlies much of `java.util.concurrent`'s high-performance internals, making it foundational knowledge for truly understanding modern Java concurrency.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|CAS|Atomic hardware instruction: compare, then swap if matched|
|Used by|All `java.util.concurrent.atomic` classes internally|
|Failure behavior|Retry (no blocking)|
|ABA problem|Value returns to original state, masking intermediate changes|
|Fix for ABA|`AtomicStampedReference` (versioned CAS)|

---

## Keywords

CAS, Compare-And-Swap, Lock-Free, ABA Problem, AtomicStampedReference, Retry Loop, Hardware Instruction

---

## Related Notes

[[30 - Atomic Variables]] [[26 - Race Condition]] [[52 - ConcurrentHashMap]] [[62 - Java Memory Model]]

---

## Practice Exercises

**Easy:**

1. Manually implement an atomic increment using `AtomicInteger.compareAndSet()` in an explicit retry loop, rather than calling `incrementAndGet()` directly.
2. Log how many retries occur (roughly) under light concurrent load, using a counter incremented on each failed CAS attempt.

**Medium:** 3. Reproduce this note's `AtomicStampedReference` ABA demonstration, and confirm the stale-stamp CAS correctly fails. 4. Implement a simple lock-free "single-slot mailbox" (an `AtomicReference<String>` that only accepts a new message if currently empty, using `compareAndSet(null, message)`).

**Hard:** 5. Research and implement a minimal lock-free stack (push/pop) using `AtomicReference` for the top-of-stack pointer, and analyze whether/how the ABA problem could theoretically affect it.

---

## Mini Project

**Project: CAS Contention Visualizer**

Build a small console application that:

1. Implements a manual CAS-based counter (as in this note's Code Example), tracking and logging the number of failed CAS attempts (retries) per thread.
2. Runs the experiment with varying thread counts (2, 10, 50), recording total retries and total time for each configuration.
3. Prints a summary showing how retry counts and total time scale as contention increases.
4. Includes a brief written reflection on when this retry overhead might justify switching to a lock-based or striped (`LongAdder`-style) approach instead.

**Goal:** Build direct, empirical insight into CAS retry behavior under varying contention, connecting the abstract mechanism to observable, measured performance characteristics.

---

## Common Interview Trap

**Trap Question:** _"Since CAS never blocks and just retries on failure, is a CAS-based counter guaranteed to always be faster than a `synchronized` counter, no matter how many threads are contending for it?"_

The trap is assuming lock-free always means strictly better performance in all conditions. The correct, nuanced answer: **not necessarily** — under **very high contention** (many threads simultaneously hammering the same atomic value), CAS-based approaches can suffer from a high rate of failed attempts and retries, along with significant CPU cache-line contention across cores, which can actually degrade throughput. This is precisely the scenario that motivated the creation of more advanced constructs like `LongAdder`, which internally spreads contention across multiple internal cells (a technique called striping) specifically to reduce this problem. For most everyday, low-to-moderate contention scenarios, CAS-based atomics do outperform lock-based approaches — but "CAS is always faster" is an oversimplification that a deeper understanding correctly challenges.

---

## Frequently Asked Questions

**Q: Is CAS unique to Java, or a general hardware/computer science concept?** A: It's a general, widely-used hardware-level concept, available on virtually all modern CPU architectures (x86, ARM, etc.) — Java's atomic classes simply expose this underlying hardware capability through a convenient, safe API.

**Q: What Java mechanism historically exposed raw CAS to Java code, before more modern APIs?** A: `sun.misc.Unsafe` was historically used internally by the JDK to access CAS and other low-level operations; modern Java (9+) provides the more standardized, safer `java.lang.invoke.VarHandle` API for similar low-level access, though most application developers never need to use either directly, relying instead on the higher-level atomic classes.

**Q: Does every `java.util.concurrent` class use CAS internally?** A: Not every single one, but a very significant portion of the high-performance, low-contention-optimized classes (atomic classes, `ConcurrentHashMap`, `ConcurrentLinkedQueue`, and others) rely on CAS as a core building block, sometimes combined with limited, fine-grained locking for specific edge cases.

**Q: Is understanding CAS necessary for everyday Java development, or only for framework/library authors?** A: For everyday application development, you can use atomic classes effectively without deep CAS knowledge — but understanding it deeply (as this note covers) significantly strengthens your ability to reason about performance, correctness, and to succeed in more rigorous technical interviews.

---

## Revision Checklist

- [ ] I can explain what CAS does and why it's atomic at the hardware level.
- [ ] I understand how Java's atomic classes use CAS internally via a retry loop.
- [ ] I can explain the ABA problem with a concrete example.
- [ ] I know how `AtomicStampedReference` solves the ABA problem.
- [ ] I understand the trade-offs between CAS-based and lock-based approaches under varying contention levels.

---

## Flashcards

**Q:** What does CAS stand for? **A:** Compare-And-Swap.

**Q:** Is CAS a software-level or hardware-level mechanism? **A:** Hardware-level — a single, atomic CPU instruction.

**Q:** What happens when a CAS operation fails? **A:** The calling code (typically) retries — reading the current value again and attempting the operation once more, with no blocking involved.

**Q:** What is the ABA problem? **A:** A value changes from A to B and back to A, making a simple CAS check incorrectly think nothing happened in between.

**Q:** How does `AtomicStampedReference` solve the ABA problem? **A:** By pairing the value with a version/stamp number that changes on every update, so even if the value returns to its original state, the stamp reveals that something happened.

---

## References

- Oracle Java Tutorials — Atomic Access: https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
- `java.util.concurrent.atomic.AtomicInteger` API Documentation
- `java.util.concurrent.atomic.AtomicStampedReference` API Documentation
- `java.lang.invoke.VarHandle` API Documentation (modern low-level CAS access)