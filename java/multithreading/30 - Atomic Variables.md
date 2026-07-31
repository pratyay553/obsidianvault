# Atomic Variables

## Objective

By the end of this note, you will understand:

- What the `java.util.concurrent.atomic` package provides, and why it exists
- How atomic classes achieve thread safety WITHOUT using locks
- How to use `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, and `AtomicReference` correctly
- When atomic classes are the right choice versus `synchronized`

This note builds on [[26 - Race Condition]] and [[29 - Volatile]] — atomic classes solve exactly the "compound operation needs to be atomic" gap that plain `volatile` cannot fill, and they preview the CAS mechanism detailed fully in [[31 - CAS]].

---

## Why should I learn this?

For the extremely common case of "I just need a thread-safe counter" (or similar simple, single-variable operations), reaching for a full `synchronized` block is often more machinery than necessary. The `java.util.concurrent.atomic` package provides purpose-built, lock-free classes that are simpler to use and often faster than hand-rolled synchronization for exactly these common cases. Every experienced Java developer reaches for `AtomicInteger` reflexively for simple shared counters — this is one of the most practically useful, everyday tools in this entire vault.

---

## Beginner Explanation

Recall from [[29 - Volatile]]: a `volatile int` guarantees visibility but not atomicity — `count++` can still lose updates. `AtomicInteger` solves this **exact** problem by providing a special counter class with built-in methods (like `incrementAndGet()`) that are **guaranteed atomic** — the entire "read, modify, write" sequence happens as one indivisible operation, with no possibility of another thread interleaving in the middle.

Think of it like a specially designed turnstile at a stadium entrance that can only let **one** person through at a time, no matter how many people push on it simultaneously — the "check if there's room, then let someone through" logic is built directly into the turnstile's mechanism itself, so there's no way for two people to squeeze through in a way that corrupts the count.

---

## Real World Analogy

**Warehouse Analogy**

Imagine a warehouse where, instead of workers manually reading a shared paper tally sheet, writing a new number, and putting it back (risking the read-modify-write race from [[22 - Synchronization]]), the warehouse installs an **electronic counter machine**. Workers simply press a button labeled "+1," and the machine's own internal circuitry guarantees the increment happens correctly, even if 20 workers press the button at the exact same instant — the machine's hardware design makes each press an atomic, indivisible operation, with no possibility of two presses interfering with each other.

`AtomicInteger` and its relatives are exactly this "purpose-built machine" for common atomic operations — you don't have to manually implement the correct locking; the class's internal implementation guarantees it.

---

## Technical Explanation

### The `java.util.concurrent.atomic` Package

Key classes:

- `AtomicInteger` — an `int` wrapper with atomic operations.
- `AtomicLong` — a `long` wrapper with atomic operations.
- `AtomicBoolean` — a `boolean` wrapper with atomic operations.
- `AtomicReference<V>` — a generic object reference wrapper with atomic operations.

### Common `AtomicInteger` Operations

```java
AtomicInteger counter = new AtomicInteger(0);

counter.get();                  // read current value
counter.set(10);                // set to a specific value
counter.incrementAndGet();      // atomically ++value, returns NEW value
counter.getAndIncrement();      // atomically value++, returns OLD value
counter.addAndGet(5);           // atomically value += 5, returns NEW value
counter.decrementAndGet();      // atomically --value, returns NEW value
counter.compareAndSet(10, 20);  // atomically: if current value == 10, set to 20; returns true if successful
```

### How Atomic Classes Achieve Thread Safety WITHOUT Locks

This is the key distinguishing feature: `AtomicInteger` and its relatives are implemented using a low-level hardware instruction called **Compare-And-Swap (CAS)** — fully detailed in [[31 - CAS]] — rather than using `synchronized`/monitor locks internally. This means:

- **No thread ever blocks** waiting for another thread to finish (no `BLOCKED` state involved).
- Under contention, a thread might need to **retry** its operation a few times (if another thread's CAS "wins" first), but it's never forced to wait passively.
- This is generally significantly **faster** than lock-based synchronization under moderate contention, since there's no context-switching overhead from blocking/unblocking threads (see [[12 - Context Switching]]).

### `AtomicReference` — Atomic Operations on Object References

```java
AtomicReference<String> configRef = new AtomicReference<>("initial-config");

configRef.set("updated-config");                          // atomic write
String current = configRef.get();                          // atomic read
boolean success = configRef.compareAndSet("initial-config", "new-config"); // atomic conditional update
```

This is commonly used for "atomically swap the entire current configuration/state object" patterns — allowing readers to always see either the complete old state or the complete new state, never a partially-updated mix.

### `compareAndSet()` — The Building Block for Custom Atomic Logic

```java
AtomicInteger value = new AtomicInteger(10);

// Only update if the current value matches what we expect
boolean updated = value.compareAndSet(10, 20); // true, since current value was indeed 10
boolean updatedAgain = value.compareAndSet(10, 30); // false, since current value is now 20, not 10
```

This method is the foundation that all the other convenience methods (`incrementAndGet()`, etc.) are built on top of internally, and it's also directly useful for implementing custom atomic logic beyond the built-in convenience methods.

---

## Internal Working

Step-by-step of how `incrementAndGet()` works internally (simplified conceptual model):

1. The thread reads the current value (say, `5`).
2. It computes the new value (`6`).
3. It calls the CAS hardware instruction: "if the value is still `5`, atomically set it to `6`; otherwise, tell me it failed."
4. **If successful** (no other thread changed the value in between steps 1 and 3): the operation is done, and `6` is returned.
5. **If it failed** (another thread changed the value in between): the thread **retries** the entire sequence — reads the new current value, computes a new target, tries CAS again — looping until it succeeds.

This retry loop happens entirely within the `AtomicInteger` class's internal implementation — from the calling code's perspective, `incrementAndGet()` simply "just works" atomically, with all this retry machinery hidden away.

---

## Memory Diagram

```
        AtomicInteger internal value (in memory)
   -----------------------------------------------
   |  value: 5                                     |
   -----------------------------------------------
        ^                              ^
        |                              |
   Thread A: incrementAndGet()    Thread B: incrementAndGet()
   1. reads 5                     1. reads 5
   2. computes 6
   3. CAS(expected=5, new=6)
      SUCCEEDS -> value = 6
                                  2. computes 6 (based on its own read of 5)
                                  3. CAS(expected=5, new=6)
                                     FAILS! (value is now 6, not 5)
                                  4. RETRIES: reads 6, computes 7
                                  5. CAS(expected=6, new=7)
                                     SUCCEEDS -> value = 7

   Final value: 7 (correct! both increments counted)
```

---

## Execution Flow

```
Thread calls atomicInt.incrementAndGet()
        |
        v
Read current value
        |
        v
Compute new value (current + 1)
        |
        v
Attempt CAS(expected=current, new=computed)
        |
   -----------------------------
   |                           |
 SUCCESS                     FAILURE (another thread changed it)
   |                           |
   v                           v
Return new value          RETRY: go back to "Read current value"
                           (loop until success)
```

---

## Code Example

```java
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;

public class AtomicVariablesDemo {

    public static void main(String[] args) throws InterruptedException {

        // --- AtomicInteger: fixing the classic lost-update problem ---
        AtomicInteger counter = new AtomicInteger(0);

        Runnable incrementTask = () -> {
            for (int i = 0; i < 100_000; i++) {
                counter.incrementAndGet(); // atomic, no lock needed
            }
        };

        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++) {
            threads[i] = new Thread(incrementTask);
        }
        for (Thread t : threads) t.start();
        for (Thread t : threads) t.join();

        System.out.println("AtomicInteger result: " + counter.get() + " (expected 1000000, should be EXACT)");

        // --- compareAndSet: implementing custom atomic logic ---
        AtomicInteger stock = new AtomicInteger(5);

        Runnable purchaseTask = () -> {
            while (true) {
                int current = stock.get();
                if (current <= 0) {
                    System.out.println(Thread.currentThread().getName() + " could not purchase: out of stock.");
                    return;
                }
                if (stock.compareAndSet(current, current - 1)) {
                    System.out.println(Thread.currentThread().getName() + " purchased! Remaining: " + (current - 1));
                    return;
                }
                // else: someone else changed it first, retry the loop
            }
        };

        Thread[] buyers = new Thread[8]; // MORE buyers than stock (5) -- some should fail
        for (int i = 0; i < 8; i++) {
            buyers[i] = new Thread(purchaseTask, "Buyer-" + i);
        }
        for (Thread t : buyers) t.start();
        for (Thread t : buyers) t.join();

        System.out.println("Final stock: " + stock.get() + " (expected 0, never negative)");
    }
}
```

### Line-by-line explanation

1. The `AtomicInteger counter` demo directly parallels the buggy `volatile` counter example from [[29 - Volatile]] — but using `incrementAndGet()` instead of a manual `count++`, this version **reliably** produces exactly `1,000,000`, every single run.
2. The `stock` demo implements custom atomic "check-then-act" logic (purchase only if stock remains) using `compareAndSet()` in a retry loop — this is the idiomatic pattern for building custom atomic logic beyond the built-in convenience methods, and correctly prevents the stock count from ever going negative, even with more buyer threads (8) than available stock (5).

---

## Dry Run

**AtomicInteger counter demo:**

```
AtomicInteger result: 1000000 (expected 1000000, should be EXACT)
```

(Reliably correct, every single run.)

**Stock purchase demo (order of buyer success/failure varies, but total is always correct):**

```
Buyer-3 purchased! Remaining: 4
Buyer-0 purchased! Remaining: 3
Buyer-1 purchased! Remaining: 2
Buyer-5 purchased! Remaining: 1
Buyer-2 purchased! Remaining: 0
Buyer-4 could not purchase: out of stock.
Buyer-6 could not purchase: out of stock.
Buyer-7 could not purchase: out of stock.
Final stock: 0 (expected 0, never negative)
```

---

## Common Mistakes

1. **Using non-atomic `int`/`Integer` fields with manual synchronization when `AtomicInteger` would be simpler and often faster.**
    
    - For simple counter/accumulator use cases, reaching for `AtomicInteger` first is generally the better default than hand-rolling `synchronized` blocks.
2. **Forgetting that individual atomic operations are atomic, but a SEQUENCE of multiple atomic calls is NOT automatically atomic as a whole.**
    
    - E.g., `if (counter.get() < 10) { counter.incrementAndGet(); }` — the `get()` and `incrementAndGet()` are each individually atomic, but the **combination** (check-then-act) is not, reintroducing exactly the check-then-act race from [[26 - Race Condition]]. Use `compareAndSet()` in a retry loop (as shown in this note's stock example) for genuinely atomic compound custom logic.
3. **Using `AtomicReference` but forgetting that it only guarantees atomicity of the reference swap itself, not the referenced object's internal state.**
    
    - If the object being pointed to is itself mutable and accessed/modified separately by multiple threads, `AtomicReference` alone doesn't protect that internal mutability.
4. **Assuming atomic classes are always faster than `synchronized` under all conditions.**
    
    - Under **very high contention** (many threads constantly retrying CAS operations), atomic classes' retry-loop behavior can sometimes perform worse than a well-designed lock-based approach — though for most typical/moderate contention levels, atomics are the faster, simpler choice.

---

## Production Usage

- **Request counters, metrics, and monitoring statistics** in high-throughput servers are almost universally implemented using `AtomicLong`/`AtomicInteger` rather than `synchronized` counters.
- **Unique ID generators** (e.g., generating sequential IDs for in-memory objects) commonly use `AtomicLong.incrementAndGet()`.
- **Circuit breaker and rate limiter implementations** frequently use `AtomicInteger`/`AtomicReference` internally for their core state tracking.
- **Lock-free data structure implementations** (like `ConcurrentLinkedQueue`, `ConcurrentHashMap`'s internals) are built using CAS-based atomic primitives as their foundational building blocks (see [[31 - CAS]], [[52 - ConcurrentHashMap]]).

---

## Interview Questions

**Beginner:**

1. What does `AtomicInteger` provide that a plain `volatile int` does not?
2. Name three common `AtomicInteger` methods and what they do.
3. Do atomic classes use locks internally?

**Intermediate:** 4. How does `compareAndSet()` work, and why is it the foundation for the other `AtomicInteger` methods? 5. Is a sequence of two separate atomic operations (e.g., `get()` then `incrementAndGet()`) automatically atomic as a combined unit? 6. What happens internally when a CAS operation fails?

**Advanced:** 7. Explain, step by step, how `incrementAndGet()` handles contention between two threads without ever blocking either one. 8. When might a `synchronized`-based approach actually outperform an atomic-class-based approach?

**Production-Level:** 9. Describe a real production scenario where you'd choose `AtomicLong` over a `synchronized` counter, and explain why. 10. How would you implement custom atomic "check-then-act" business logic (like the stock purchase example) using `compareAndSet()`?

---

## Follow-up Questions

- Does `AtomicInteger` extend `Number`, and can it be used directly wherever an `int` is expected? _(It extends `Number` and provides conversion methods like `.intValue()`, but it is NOT a drop-in replacement for a primitive `int` in all contexts — you must call `.get()` or similar to extract the primitive value where needed.)_
- Is there an `AtomicDouble` or `AtomicFloat` in the standard `java.util.concurrent.atomic` package? _(No — the standard package covers `int`, `long`, `boolean`, and object references; floating-point atomics aren't included directly, though `AtomicReference<Double>` or third-party libraries can be used if genuinely needed, keeping in mind the boxing overhead.)_
- Can multiple different atomic operations on DIFFERENT `AtomicInteger` instances be coordinated to update together atomically, as a single transaction? _(Not directly with basic atomic classes — this would require additional coordination, like using a shared lock across the group, or a more advanced/transactional approach; the atomic classes themselves each only guarantee atomicity for their own single value.)_

---

## Best Practices

- Default to atomic classes (`AtomicInteger`, `AtomicLong`, etc.) for simple, single-variable thread-safe counters/accumulators — they're simpler and often faster than hand-rolled `synchronized` blocks for this common case.
- Use `compareAndSet()` in a retry loop for custom atomic compound logic beyond the built-in convenience methods (as demonstrated in the stock purchase example).
- Remember that a sequence of multiple atomic calls is not automatically atomic as a whole — design accordingly.
- Reach for `synchronized`/explicit locks instead when you need to atomically coordinate updates across **multiple** related pieces of shared state together, not just a single value.

---

## Performance Notes

- Under low-to-moderate contention, atomic classes generally outperform `synchronized` equivalents, since there's no blocking/context-switching overhead.
- Under extremely high contention, the CAS retry loop can cause many wasted retries, and in some such cases, a well-tuned lock-based approach might perform comparably or better — but this is a less common scenario for most typical application workloads.
- Atomic classes avoid the `BLOCKED` thread-state pileups that heavily contended `synchronized` blocks can produce, which is often a significant advantage for overall system responsiveness under load.

---

## Comparison Table

|Aspect|`AtomicInteger`|`synchronized` counter|
|---|---|---|
|Uses locks?|No (CAS-based)|Yes|
|Can threads block?|No (retry instead)|Yes|
|Typical performance (low/moderate contention)|Faster|Slower|
|Typical performance (extreme contention)|Can degrade (many retries)|More predictable, but still has blocking overhead|
|Ease of use for simple counters|Very simple|More boilerplate (lock object, synchronized block)|

---

## Summary

- The `java.util.concurrent.atomic` package provides classes (`AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference`) offering genuinely atomic operations without using locks.
- Internally, these classes use the CAS (Compare-And-Swap) hardware instruction, retrying automatically on contention rather than blocking (see [[31 - CAS]] for full detail).
- Individual atomic operations are atomic, but sequences of multiple separate atomic calls are not automatically atomic as a combined unit — use `compareAndSet()` in a retry loop for custom atomic compound logic.
- Atomic classes are the idiomatic, often-faster default choice for simple shared counters and similar single-variable thread-safe needs in modern Java code.

---

## Cheat Sheet

|Method|Behavior|
|---|---|
|`get()` / `set(v)`|Atomic read / write|
|`incrementAndGet()` / `getAndIncrement()`|Atomic `++value`, returning new/old value respectively|
|`addAndGet(n)`|Atomic `value += n`, returns new value|
|`compareAndSet(expected, new)`|Atomic conditional update; returns true if successful|
|Internal mechanism|CAS (Compare-And-Swap), no locks|

---

## Keywords

AtomicInteger, AtomicLong, AtomicReference, compareAndSet, CAS, Lock-Free, Retry Loop, java.util.concurrent.atomic

---

## Related Notes

[[22 - Synchronization]] [[26 - Race Condition]] [[29 - Volatile]] [[31 - CAS]] [[52 - ConcurrentHashMap]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `AtomicInteger` counter demo and confirm it always produces exactly the expected total.
2. Use `AtomicBoolean` to implement a simple, correctly-working "run once" flag shared across multiple threads (only the first thread to successfully `compareAndSet(false, true)` should proceed with some one-time action).

**Medium:** 3. Reproduce the stock purchase demo, and experiment with different ratios of buyers to available stock, confirming the final stock count is always correct and never negative. 4. Implement a custom atomic "maximum value tracker" using `AtomicInteger.compareAndSet()` in a retry loop — a shared value that should always hold the highest value any thread has ever reported, updated atomically and correctly under concurrent access.

**Hard:** 5. Benchmark an `AtomicInteger`-based counter against a `synchronized`-based counter under varying levels of thread contention (e.g., 2, 10, 50, 200 threads), and analyze at what point (if any) the atomic version's performance advantage narrows or reverses on your machine.

---

## Mini Project

**Project: Lock-Free Inventory System**

Build a small console application that:

1. Uses `AtomicInteger` to track inventory levels for several different products (e.g., a `Map<String, AtomicInteger>`).
2. Simulates many concurrent "purchase" threads attempting to buy various products, using `compareAndSet()`-based retry logic to ensure inventory never goes negative.
3. Simulates concurrent "restock" threads that atomically add to inventory levels.
4. Reports final inventory levels for all products, confirming correctness under heavy concurrent load, and prints how many purchase attempts succeeded versus failed due to insufficient stock.

**Goal:** Build a realistic, business-relevant, lock-free concurrent system using atomic variables as the sole synchronization mechanism — reinforcing both the practical API and the underlying retry-loop pattern.

---

## Common Interview Trap

**Trap Question:** _"If I write `if (atomicCounter.get() < 100) { atomicCounter.incrementAndGet(); }`, is this code free of race conditions, since both `get()` and `incrementAndGet()` are individually atomic?"_

The trap is assuming that because each individual call is atomic, the combination is too. The correct answer: **no** — this is a classic check-then-act race condition (see [[26 - Race Condition]]). Between the `get()` call and the `incrementAndGet()` call, another thread could also read a value below 100 and also increment, potentially pushing the counter above 100 despite the intended limit. Each individual atomic operation is safe in isolation, but the **sequence** of two separate atomic calls is not automatically atomic as a whole. The correct fix is to implement this logic using `compareAndSet()` in a retry loop (checking and conditionally updating as a single atomic step), exactly like the stock purchase pattern demonstrated in this note.

---

## Frequently Asked Questions

**Q: Is `AtomicInteger` a subclass of `Integer`?** A: No — it's a separate class that internally wraps an `int` value and provides atomic operations; it is not interchangeable with `Integer` or `int` without explicit conversion (`.get()`, `.intValue()`, etc.).

**Q: Can `AtomicReference` be used to implement a simple, thread-safe "latest value" cache?** A: Yes — this is a common, idiomatic use case: atomically swapping in a new immutable snapshot object whenever the underlying data changes, allowing readers to always see a complete, consistent snapshot.

**Q: Do atomic classes provide any ordering guarantees beyond just atomicity?** A: Yes — they provide the same visibility/ordering guarantees as `volatile` (see [[29 - Volatile]]) in addition to atomicity for their supported operations, since they're built on top of similar underlying memory model guarantees.

**Q: Are there atomic array classes, for atomic operations on individual array elements?** A: Yes — `AtomicIntegerArray`, `AtomicLongArray`, and `AtomicReferenceArray` exist for exactly this purpose, providing atomic operations on individual elements of an array without needing a full array-wide lock.

---

## Revision Checklist

- [ ] I understand why `AtomicInteger` solves the problem that plain `volatile int` cannot.
- [ ] I know the core `AtomicInteger` methods and what they do.
- [ ] I understand that atomic classes use CAS internally, not locks.
- [ ] I understand that a sequence of multiple atomic calls is not automatically atomic as a whole.
- [ ] I know how to use `compareAndSet()` in a retry loop to implement custom atomic logic.

---

## Flashcards

**Q:** What problem does `AtomicInteger` solve that plain `volatile int` cannot? **A:** It provides genuinely atomic compound operations (like increment), not just visibility.

**Q:** Do atomic classes use locks internally? **A:** No — they use the CAS (Compare-And-Swap) hardware instruction, retrying on contention instead of blocking.

**Q:** Is a sequence of two separate atomic calls automatically atomic as a whole? **A:** No — this is still a potential check-then-act race condition; use `compareAndSet()` in a retry loop for genuinely atomic custom compound logic.

**Q:** What does `compareAndSet(expected, new)` do? **A:** Atomically sets the value to `new` only if the current value equals `expected`, returning true if successful.

**Q:** Name the four main classes in `java.util.concurrent.atomic`. **A:** `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference`.

---

## References

- Oracle Java Tutorials — Atomic Access: https://docs.oracle.com/javase/tutorial/essential/concurrency/atomic.html
- `java.util.concurrent.atomic` Package Documentation
- `java.util.concurrent.atomic.AtomicInteger` API Documentation