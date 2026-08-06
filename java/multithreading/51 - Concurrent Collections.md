# Concurrent Collections

## Objective

By the end of this note, you will understand:

- Why the standard `java.util` collections (`HashMap`, `ArrayList`, etc.) are unsafe for concurrent use
- What "fail-fast" iteration means and why it causes `ConcurrentModificationException`
- The overall landscape of `java.util.concurrent`'s collection alternatives
- How to choose the right concurrent collection for a given need

This note opens **Section 9: Concurrent Collections & Coordination Utilities**, providing the map of the territory before diving into specific classes like [[52 - ConcurrentHashMap]] and [[53 - CopyOnWriteArrayList]].

---

## Why should I learn this?

You've now seen, repeatedly throughout this vault (Sections 4, 6, 8), that using standard collections (`HashMap`, `ArrayList`) from multiple threads without external synchronization causes race conditions and corruption. `java.util.concurrent` provides purpose-built, thread-safe collection implementations that solve this — but choosing the **wrong** one, or misunderstanding what "thread-safe" actually guarantees for a given collection, is itself a common source of bugs. This note builds the mental map before the deep dives that follow.

---

## Beginner Explanation

Standard Java collections (`ArrayList`, `HashMap`, `HashSet`) were designed assuming **single-threaded** access — their internal implementation (array resizing, hash bucket restructuring) simply isn't safe if multiple threads read and write concurrently. It's like a filing cabinet designed for one person to use at a time: if two people try to reorganize it simultaneously, papers get lost or misfiled.

`java.util.concurrent` provides an entire family of **thread-safe** collection implementations — filing cabinets specifically engineered to handle multiple people accessing them at once, safely, without external locking required.

---

## Real World Analogy

**Library Catalog Analogy**

Imagine an old library with a single, physical card catalog drawer — if two librarians try to add cards to it at the exact same moment, they could physically collide, misfile cards, or damage the drawer's mechanism. This is a standard `HashMap`/`ArrayList` — not designed for simultaneous access.

A modern library's **digital catalog system**, by contrast, is specifically engineered from the ground up to handle many librarians (and patrons) simultaneously searching, adding, and updating records — with careful internal design ensuring no corruption occurs, no matter how many people use it at once. This is exactly what `java.util.concurrent`'s collections provide.

---

## Technical Explanation

### Why Standard Collections Fail Under Concurrent Access

**1. Data corruption from concurrent structural modification**

```java
Map<String, Integer> map = new HashMap<>(); // NOT thread-safe

// Thread A and Thread B both calling map.put(...) concurrently
// can corrupt HashMap's internal bucket/linked-list structure,
// potentially causing lost entries, infinite loops (in older JDK
// versions), or other undefined behavior.
```

**2. `ConcurrentModificationException` — "fail-fast" iteration**

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));

for (Integer i : list) {
    if (i == 2) {
        list.remove(i); // throws ConcurrentModificationException!
    }
}
```

Standard collections use a **fail-fast** iterator design: they maintain an internal "modification count," and if the collection is structurally modified **during** iteration (by any means other than the iterator's own `remove()`), the next call to the iterator throws `ConcurrentModificationException` — a deliberate, defensive mechanism to **catch bugs early** rather than silently producing corrupted or unpredictable results. This applies even in **single-threaded** code (as shown above), but becomes an even more common, harder-to-predict issue in multithreaded code, where one thread might modify a collection while another is iterating over it.

### The `java.util.concurrent` Collection Landscape

|Category|Examples|Approach|
|---|---|---|
|Concurrent Maps|`ConcurrentHashMap` (see [[52 - ConcurrentHashMap]])|Fine-grained internal locking / lock-free techniques|
|Copy-on-Write Lists/Sets|`CopyOnWriteArrayList` (see [[53 - CopyOnWriteArrayList]])|Full copy on every write; safe, snapshot-based iteration|
|Blocking Queues|`ArrayBlockingQueue`, `LinkedBlockingQueue` (see [[43 - BlockingQueue]])|Built-in blocking put/take coordination|
|Concurrent Queues (non-blocking)|`ConcurrentLinkedQueue`|Lock-free (CAS-based, see [[31 - CAS]])|
|Concurrent Navigable Structures|`ConcurrentSkipListMap`, `ConcurrentSkipListSet`|Lock-free, sorted structures|

### A Simple, Universal Alternative (For Contrast): `Collections.synchronizedX()`

```java
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
```

This wraps a standard collection with **synchronized** methods (using a single internal lock for every operation) — it's thread-safe for **individual** operations, but:

- Provides **no** better performance under contention than a single `synchronized` block would (every operation blocks every other operation).
- **Still requires manual external synchronization for iteration** (you must wrap the iteration itself in a `synchronized` block on the same lock, or you can still get `ConcurrentModificationException`).
- Is generally **inferior** to the purpose-built `java.util.concurrent` alternatives, which are specifically engineered for better concurrent performance (e.g., `ConcurrentHashMap`'s finer-grained internal locking versus one single lock for the whole structure).

### The Core Distinction: "Thread-Safe" Doesn't Always Mean "Compound Operations Are Atomic"

Exactly as covered in [[26 - Race Condition]]'s `ConcurrentHashMap` example: individual operations on any `java.util.concurrent` collection are thread-safe, but **sequences** of multiple operations are not automatically atomic unless you specifically use a purpose-built atomic compound method (`putIfAbsent()`, `computeIfAbsent()`, etc.). This principle applies across the **entire** concurrent collections family, not just `ConcurrentHashMap` — it's worth internalizing as a universal rule for this whole category of class.

---

## Internal Working

Conceptually, how concurrent collections differ from `Collections.synchronizedX()` wrappers:

1. A `synchronizedList`/`synchronizedMap` uses **one single lock** for the entire collection — every operation, regardless of what part of the collection it touches, contends for that same lock.
2. Purpose-built concurrent collections use **more sophisticated internal strategies**: `ConcurrentHashMap` uses fine-grained locking (or lock-free CAS operations, in modern JDK versions) scoped to specific internal segments/buckets, letting unrelated operations proceed genuinely concurrently; `CopyOnWriteArrayList` avoids locking readers entirely by having writers create an entirely new internal array copy.
3. This internal sophistication is exactly why purpose-built concurrent collections generally **outperform** simple synchronized wrappers under real concurrent load, despite providing the same fundamental thread-safety guarantee for individual operations.

---

## Memory Diagram

```
   Collections.synchronizedMap(map)          ConcurrentHashMap
   -----------------------------            -----------------------------
   |  ONE lock for the WHOLE map |            |  Multiple internal locks/  |
   |  (every op contends here)   |            |  CAS-based segments        |
   -----------------------------            -----------------------------
   Thread A put()  --BLOCKS-->  Thread B put()   Thread A put() (bucket 1)
                                                   Thread B put() (bucket 5)
                                                   -- run CONCURRENTLY, no
                                                      contention if different
                                                      buckets
```

---

## Execution Flow

```
Need a thread-safe collection
        |
        v
Is it a Map? --YES--> ConcurrentHashMap (almost always the right choice)
        | NO
        v
Is it a List/Set with MANY MORE reads than writes,
and writes are relatively rare? --YES--> CopyOnWriteArrayList/Set
        | NO
        v
Do you need blocking put/take semantics (producer-consumer)? --YES--> BlockingQueue implementation
        | NO
        v
Do you need a simple, lock-free, non-blocking queue? --YES--> ConcurrentLinkedQueue
        | NO
        v
Do you need a sorted, navigable structure? --YES--> ConcurrentSkipListMap/Set
```

---

## Code Example

```java
import java.util.*;
import java.util.concurrent.*;

public class ConcurrentCollectionsDemo {

    public static void main(String[] args) throws InterruptedException {

        // 1. Demonstrate ConcurrentModificationException with a standard collection
        List<Integer> unsafeList = new ArrayList<>(List.of(1, 2, 3, 4, 5));
        try {
            for (Integer i : unsafeList) {
                if (i == 3) {
                    unsafeList.remove(i); // structural modification during iteration
                }
            }
        } catch (ConcurrentModificationException e) {
            System.out.println("Caught expected ConcurrentModificationException on standard ArrayList");
        }

        // 2. Standard HashMap corrupted under concurrent writes (illustrative -- may or may not throw/corrupt visibly every run)
        Map<Integer, Integer> unsafeMap = new HashMap<>();
        Runnable unsafeWriter = () -> {
            for (int i = 0; i < 10_000; i++) {
                unsafeMap.put(i, i);
            }
        };
        Thread t1 = new Thread(unsafeWriter);
        Thread t2 = new Thread(unsafeWriter);
        t1.start(); t2.start();
        t1.join(); t2.join();
        System.out.println("Unsafe HashMap final size: " + unsafeMap.size() + " (expected 10000, may be corrupted/incorrect)");

        // 3. Safe: ConcurrentHashMap under the same concurrent writes
        Map<Integer, Integer> safeMap = new ConcurrentHashMap<>();
        Runnable safeWriter = () -> {
            for (int i = 0; i < 10_000; i++) {
                safeMap.put(i, i);
            }
        };
        Thread t3 = new Thread(safeWriter);
        Thread t4 = new Thread(safeWriter);
        t3.start(); t4.start();
        t3.join(); t4.join();
        System.out.println("Safe ConcurrentHashMap final size: " + safeMap.size() + " (should be EXACTLY 10000)");
    }
}
```

### Line-by-line explanation

1. The first block demonstrates the **fail-fast** iterator behavior — removing an element during a standard `for-each` loop throws `ConcurrentModificationException`, even in single-threaded code.
2. The second block demonstrates concurrent writes to a standard `HashMap` — the result is unreliable and can be smaller than expected (or, in older JDK versions, could theoretically cause more severe issues like infinite loops during resize) due to internal structural corruption.
3. The third block demonstrates the same workload against `ConcurrentHashMap`, reliably producing the correct size every time.

---

## Dry Run

```
Caught expected ConcurrentModificationException on standard ArrayList
Unsafe HashMap final size: 9847 (expected 10000, may be corrupted/incorrect)
Safe ConcurrentHashMap final size: 10000 (should be EXACTLY 10000)
```

---

## Common Mistakes

1. **Using `Collections.synchronizedX()` and assuming it's equivalent to a purpose-built concurrent collection.**
    
    - It provides thread safety for individual operations but with worse concurrent performance (single lock for everything) and still requires external synchronization for safe iteration.
2. **Iterating over a `Collections.synchronizedList()` without additional synchronization.**
    
    - Even though individual `get()`/`add()` calls are synchronized internally, iteration itself is NOT automatically safe against concurrent modification — you must manually `synchronized` the entire iteration block on the same lock object.
3. **Assuming ANY concurrent collection makes compound operations atomic automatically.**
    
    - As emphasized, only specific, purpose-built compound methods (`putIfAbsent`, `computeIfAbsent`, etc.) provide this — a plain sequence of individually-safe calls is still subject to race conditions (see [[26 - Race Condition]]).
4. **Defaulting to standard collections in multithreaded code "because it's simpler," without recognizing the concurrency requirement.**
    
    - Always explicitly consider: will this collection ever be accessed by more than one thread? If yes, a standard collection is the wrong default choice.

---

## Production Usage

- **`ConcurrentHashMap`** is, by a wide margin, the most commonly used concurrent collection in production Java — used for caches, shared lookup tables, counters, and more (full detail in [[52 - ConcurrentHashMap]]).
- **`CopyOnWriteArrayList`** is commonly used for listener/observer lists — collections that are read very frequently but modified rarely (full detail in [[53 - CopyOnWriteArrayList]]).
- **`BlockingQueue` implementations** underlie thread pool task queues and producer-consumer pipelines throughout production systems (see [[43 - BlockingQueue]], [[44 - Thread Pool]]).
- **Framework internals**: Spring, connection pool libraries, and countless other production frameworks rely heavily on this collection family internally for their own thread-safe state management.

---

## Interview Questions

**Beginner:**

1. Why are standard `java.util` collections like `HashMap` and `ArrayList` unsafe for concurrent use?
2. What is `ConcurrentModificationException`, and when is it thrown?
3. Name two classes from `java.util.concurrent` designed to be thread-safe collections.

**Intermediate:** 4. What's the difference between `Collections.synchronizedMap()` and `ConcurrentHashMap`? 5. Does a thread-safe collection automatically make compound operations (like check-then-insert) atomic? 6. Why might `Collections.synchronizedList()` still throw `ConcurrentModificationException` during iteration?

**Advanced:** 7. Explain, conceptually, why `ConcurrentHashMap` generally outperforms `Collections.synchronizedMap()` under concurrent load. 8. What is "fail-fast" iteration, and why is it a deliberate design choice rather than a bug?

**Production-Level:** 9. How would you decide between `ConcurrentHashMap`, `Collections.synchronizedMap()`, and manually synchronizing a `HashMap` yourself for a new piece of shared state? 10. Describe a real production scenario where `CopyOnWriteArrayList` would be an appropriate choice, and explain why.

---

## Follow-up Questions

- Does `ConcurrentModificationException` only happen with multiple threads? _(No — it can and does happen in purely single-threaded code, as shown in this note's first demo; it's a general "don't structurally modify a collection while iterating it (except via the iterator's own remove)" rule, made more likely/complex in multithreaded contexts.)_
- Is it ever acceptable to use `Collections.synchronizedX()` instead of a purpose-built concurrent collection? _(Yes, for simple, low-contention scenarios where the specific concurrent collection's added complexity/behavior differences aren't needed — though for most modern code, purpose-built alternatives are generally preferred.)_
- Do all `java.util.concurrent` collections have "weakly consistent" iterators (not throwing `ConcurrentModificationException`)? _(Most do — e.g., `ConcurrentHashMap`'s iterator reflects the state of the map at some point during (or since) the iteration began, without throwing an exception even if the map is modified concurrently; this is a deliberate, different trade-off from standard collections' fail-fast behavior.)_

---

## Best Practices

- Always use a purpose-built `java.util.concurrent` collection (not a standard collection, and generally not a `synchronizedX()` wrapper either) for any data structure accessed by multiple threads.
- Never assume compound operations are automatically atomic on any concurrent collection — verify you're using the correct atomic compound method for your specific need.
- Choose the specific concurrent collection based on your actual access pattern (read-heavy vs. write-heavy, blocking vs. non-blocking, ordered vs. unordered) — covered in depth in the following notes of this section.

---

## Performance Notes

- Purpose-built concurrent collections generally significantly outperform `Collections.synchronizedX()` wrappers under real concurrent load, due to finer-grained internal locking or lock-free techniques.
- The specific performance characteristics vary meaningfully between different concurrent collections (e.g., `ConcurrentHashMap`'s write-optimized design versus `CopyOnWriteArrayList`'s read-optimized design) — choosing the right one for your actual read/write ratio matters significantly (fully explored in the following notes).

---

## Comparison Table

|Approach|Thread-Safe Individual Ops?|Safe Iteration?|Performance Under Contention|
|---|---|---|---|
|Standard collection (`HashMap`, etc.)|No|No|N/A (unsafe)|
|`Collections.synchronizedX()`|Yes|No (needs manual external sync)|Poor (single lock)|
|Purpose-built concurrent collection|Yes|Often yes (weakly consistent iterators)|Good (fine-grained/lock-free)|

---

## Summary

- Standard `java.util` collections are not thread-safe and can suffer both silent data corruption and `ConcurrentModificationException` under concurrent (or even certain single-threaded) modification-during-iteration scenarios.
- `java.util.concurrent` provides a family of purpose-built, thread-safe collections: `ConcurrentHashMap`, `CopyOnWriteArrayList`, various `BlockingQueue` implementations, `ConcurrentLinkedQueue`, and `ConcurrentSkipListMap`/`Set`.
- `Collections.synchronizedX()` wrappers provide basic thread safety but with worse performance and still-manual iteration synchronization requirements — generally inferior to purpose-built alternatives.
- Thread-safety of individual operations does NOT automatically mean compound operations are atomic — this universal rule applies across the entire concurrent collections family.
- Choosing the right specific concurrent collection depends on your actual access pattern, covered in the notes that follow in this section.

---

## Cheat Sheet

|Need|Use|
|---|---|
|General-purpose thread-safe map|`ConcurrentHashMap`|
|Read-heavy list/set, rare writes|`CopyOnWriteArrayList`/`CopyOnWriteArraySet`|
|Producer-consumer with blocking|`ArrayBlockingQueue`/`LinkedBlockingQueue`|
|Simple non-blocking queue|`ConcurrentLinkedQueue`|
|Sorted, thread-safe structure|`ConcurrentSkipListMap`/`Set`|

---

## Keywords

Concurrent Collections, ConcurrentModificationException, Fail-Fast Iterator, Collections.synchronizedX, java.util.concurrent

---

## Related Notes

[[26 - Race Condition]] [[43 - BlockingQueue]] [[52 - ConcurrentHashMap]] [[53 - CopyOnWriteArrayList]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `ConcurrentModificationException` demo, and confirm the exception is thrown even in single-threaded code.
2. Reproduce the unsafe `HashMap` vs. safe `ConcurrentHashMap` concurrent-write comparison.

**Medium:** 3. Wrap a `HashMap` with `Collections.synchronizedMap()`, and demonstrate that iterating it without additional external synchronization can still throw `ConcurrentModificationException` under concurrent modification. 4. Fix the above by properly synchronizing the iteration block on the same lock object, and confirm it no longer throws.

**Hard:** 5. Build a small benchmark comparing `Collections.synchronizedMap()` against `ConcurrentHashMap` under heavy concurrent read/write load, measuring the performance difference.

---

## Mini Project

**Project: Concurrent Collection Selector Tool**

Build a small console application that:

1. Implements a simple decision-tree questionnaire (as text prompts): "Is this a Map or a List/Set?", "Read-heavy or write-heavy?", "Do you need blocking semantics?", etc.
2. Based on the answers, recommends the appropriate `java.util.concurrent` collection class, with a brief justification.
3. Includes a small demo/benchmark for at least 3 different scenarios, showing the recommended collection actually performing well for that specific access pattern.

**Goal:** Turn the collection-selection decision process from this note into an interactive, reinforced learning tool — solidifying the mapping between access patterns and the right concurrent collection choice.

---

## Common Interview Trap

**Trap Question:** _"If I wrap my `HashMap` with `Collections.synchronizedMap()`, is it now completely safe to iterate over it from multiple threads without any additional precautions?"_

The trap is assuming the wrapper's synchronization extends automatically to iteration. The correct answer: **no** — `Collections.synchronizedMap()` synchronizes each **individual** method call (like a single `get()` or `put()`), but iteration involves a **sequence** of calls to the iterator (`hasNext()`, `next()`, etc.), and the wrapper does **not** automatically synchronize this entire sequence as one atomic unit. The official Java documentation explicitly states that manual synchronization on the returned map object is required when iterating over it, precisely to avoid `ConcurrentModificationException` if another thread modifies the map during that iteration. This is a genuinely common, subtle mistake — developers often assume "it's a synchronized wrapper" means "everything about it, including iteration, is automatically safe," when in fact only individual operations are protected, and iteration requires explicit, manual, additional synchronization.

---

## Frequently Asked Questions

**Q: Is `Vector` (an older Java class) thread-safe?** A: Yes — `Vector` (and `Hashtable`) are legacy classes from Java 1.0 that use synchronized methods internally, similar in spirit to `Collections.synchronizedList()`/`synchronizedMap()`, but are now generally considered outdated; modern code should prefer purpose-built `java.util.concurrent` alternatives instead.

**Q: Do all `java.util.concurrent` collections avoid `ConcurrentModificationException` entirely?** A: Most provide "weakly consistent" iterators that don't throw this exception even under concurrent modification, instead reflecting some consistent snapshot of the data at or since iteration start — though the exact behavior can vary by specific class, so it's worth checking documentation for any collection you're using in this way.

**Q: Can you convert a standard collection into a concurrent one after the fact?** A: Not directly by "converting" — you'd typically create a new instance of the desired concurrent collection type and copy the data into it, or simply design your code from the start to use the appropriate concurrent collection wherever shared, multithreaded access is anticipated.

**Q: Is there a concurrent version of `TreeMap`/`TreeSet`?** A: Yes — `ConcurrentSkipListMap` and `ConcurrentSkipListSet` provide sorted, navigable, thread-safe alternatives, using a skip-list-based internal structure rather than a red-black tree.

---

## Revision Checklist

- [ ] I understand why standard collections are unsafe for concurrent access.
- [ ] I understand what `ConcurrentModificationException` is and when it's thrown (including in single-threaded code).
- [ ] I know the difference between `Collections.synchronizedX()` and purpose-built concurrent collections.
- [ ] I understand that thread-safety of individual operations doesn't guarantee atomicity of compound operations.
- [ ] I can map common access patterns (read-heavy, write-heavy, blocking, sorted) to the appropriate concurrent collection choice.

---

## Flashcards

**Q:** Why are standard `java.util` collections unsafe for concurrent access? **A:** They were designed assuming single-threaded access; concurrent structural modification can corrupt their internal state.

**Q:** What is "fail-fast" iteration? **A:** A deliberate design where structurally modifying a collection during iteration (other than via the iterator's own remove) throws `ConcurrentModificationException`.

**Q:** Does `Collections.synchronizedMap()` make iteration automatically safe? **A:** No — iteration still requires manual external synchronization on the same lock.

**Q:** Does a thread-safe collection make compound operations (like check-then-insert) automatically atomic? **A:** No — use purpose-built atomic compound methods like `putIfAbsent()`/`computeIfAbsent()` for that.

**Q:** What is generally the best default choice for a thread-safe map in modern Java? **A:** `ConcurrentHashMap`.

---

## References

- Oracle Java Tutorials — Concurrent Collections: https://docs.oracle.com/javase/tutorial/essential/concurrency/collections.html
- `java.util.Collections` API Documentation (`synchronizedMap`, `synchronizedList`)
- `java.util.ConcurrentModificationException` API Documentation