# Parallel Streams

## Objective

By the end of this note, you will understand:

- How `.parallelStream()` works internally, connecting directly to [[48 - Fork Join Pool]]
- When parallel streams genuinely help versus when they hurt performance
- The critical danger of using non-thread-safe operations inside a parallel stream
- Why "just add `.parallel()`" is often bad advice without careful consideration

This note closes out **Section 8: Thread Pools & Executors**, showing the highest-level, most convenient — and most easily misused — abstraction built on everything else in this section.

---

## Why should I learn this?

Parallel streams are the easiest way to accidentally introduce a subtle concurrency bug in modern Java: they look like a simple, one-word performance optimization (`.stream()` → `.parallelStream()`), but underneath, they're using the exact `ForkJoinPool` mechanics from [[48 - Fork Join Pool]], and they inherit **every** correctness concern from Sections 4-7 of this vault if you're not careful. Understanding what's really happening prevents both "adding `.parallel()` everywhere hoping for free speed" and "avoiding parallel streams entirely out of unfounded fear."

---

## Beginner Explanation

A regular Java Stream (`.stream()`) processes elements **one at a time, in order**, on a single thread. A **parallel stream** (`.parallelStream()`, or `.stream().parallel()`) instead **splits** the data into chunks and processes multiple chunks **simultaneously** across multiple threads — using exactly the fork/join, divide-and-conquer mechanism covered in [[48 - Fork Join Pool]] — then combines the results back together.

It's essentially a **one-line way to apply fork/join parallelism** to common collection-processing operations, without writing any `RecursiveTask` code yourself.

---

## Real World Analogy

**Warehouse Sorting Analogy**

Imagine sorting a huge bin of mixed recyclables into separate categories (paper, plastic, glass). A **sequential stream** is like one person going through the entire bin, item by item, sorting each one before moving to the next. A **parallel stream** is like splitting the bin into several smaller piles and having multiple people sort simultaneously, each handling their own pile, with the final sorted categories combined at the end.

This is clearly faster **if** the bin is large enough and sorting each item takes real effort — but if the bin only has 5 items, the time spent explaining the plan, splitting the pile, and gathering everyone back together might take **longer** than just sorting those 5 items yourself.

---

## Technical Explanation

### Basic Usage

```java
List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int sequentialSum = numbers.stream()
        .mapToInt(Integer::intValue)
        .sum();

int parallelSum = numbers.parallelStream()
        .mapToInt(Integer::intValue)
        .sum();
```

Both produce the same **correct** result for this simple, safe case — the difference is purely about execution strategy underneath.

### The Connection to `ForkJoinPool`

Parallel streams, **by default**, use the **common `ForkJoinPool`** (see [[48 - Fork Join Pool]]) — the exact same JVM-wide shared pool used by `CompletableFuture`'s default async methods. This has two important implications:

1. Parallel stream operations are subject to the exact same "good for CPU-bound divide-and-conquer work, bad for blocking I/O" guidance from [[48 - Fork Join Pool]].
2. Since the common pool is **shared JVM-wide**, heavy parallel stream usage in one part of your application can affect the performance of unrelated `CompletableFuture` or parallel stream operations elsewhere in the same JVM, since they're all competing for the same limited thread pool.

### When Parallel Streams Genuinely Help

- Large datasets (the splitting/coordination overhead needs to be small relative to total work).
- Computationally expensive per-element operations (not simple, cheap operations like basic addition).
- Operations that are **stateless** and **independent** per element (no shared mutable state being touched).
- A machine with multiple genuinely available CPU cores (parallel streams provide no benefit on a single-core system, and can even hurt due to overhead).

### When Parallel Streams Hurt or Are Dangerous

**1. Small datasets — overhead outweighs benefit:**

```java
List.of(1, 2, 3).parallelStream().map(x -> x * 2).toList(); // parallelism overhead is pure waste here
```

**2. Operations with shared mutable state — genuine correctness bugs:**

```java
List<Integer> results = new ArrayList<>(); // NOT thread-safe!
numbers.parallelStream().forEach(n -> results.add(n * 2)); // RACE CONDITION -- can corrupt the list or lose elements
```

This is a **direct, real-world instance** of exactly the race condition problem from [[26 - Race Condition]] — `ArrayList.add()` is not thread-safe, and multiple parallel stream threads calling it concurrently can corrupt its internal state or silently drop elements. The correct fix is to use `.collect(Collectors.toList())` (which handles thread-safety internally) instead of manually accumulating into a shared, non-thread-safe collection via `forEach`.

**3. Operations with side effects in a specific required order:**

```java
numbers.parallelStream().forEach(n -> System.out.println(n)); // order is NOT guaranteed with parallel streams!
```

Parallel streams process elements in an **unspecified order** — if your operation depends on sequential ordering (e.g., printing numbers 1 through 10 in order), a parallel stream will likely produce out-of-order results, unless you specifically use `forEachOrdered()` (which reintroduces ordering, but also reduces the parallelism benefit).

### Correct, Safe Parallel Stream Patterns

```java
// Safe: using collect() with a proper thread-safe collector
List<Integer> doubled = numbers.parallelStream()
        .map(n -> n * 2)
        .collect(Collectors.toList()); // thread-safe internally

// Safe: reduce() with a stateless, associative combining function
int sum = numbers.parallelStream()
        .reduce(0, Integer::sum); // safe: no shared mutable state, purely functional combination
```

---

## Internal Working

Step-by-step of what happens when you call `.parallelStream()...collect()`:

1. The stream source (e.g., a `List`) is recursively **split** into smaller chunks — internally, this uses a `Spliterator`, which knows how to partition the specific data structure efficiently.
2. Each chunk is processed as a separate task, submitted to the common `ForkJoinPool`, using the exact fork/join, work-stealing mechanism from [[48 - Fork Join Pool]].
3. Intermediate operations (`map`, `filter`, etc.) are applied to each chunk independently, in parallel.
4. Terminal operations (`collect`, `reduce`, `sum`, etc.) combine the per-chunk results back together — the specific combining logic depends on the terminal operation (e.g., `Collectors.toList()` has an internal, thread-safe combining strategy; `reduce()` uses your provided, ideally associative, combining function).

---

## Memory Diagram

```
        List<Integer> [1,2,3,4,5,6,7,8,9,10]
                    |
          (Spliterator splits it)
                    |
       -------------+-------------
       |                         |
   [1,2,3,4,5]               [6,7,8,9,10]
       |                         |
   (further split,           (further split,
    processed via              processed via
    ForkJoinPool workers)      ForkJoinPool workers)
       |                         |
       -------------+-------------
                    |
            Results COMBINED
            (order of combination NOT guaranteed
             unless using forEachOrdered or similar)
```

---

## Execution Flow

```
list.parallelStream()
        |
        v
Spliterator recursively splits the source
        |
        v
Chunks submitted as tasks to common ForkJoinPool
        |
        v
Intermediate ops (map/filter) applied per chunk, IN PARALLEL
        |
        v
Terminal op (collect/reduce/sum) combines results
        |
        v
Final result returned (blocking the calling thread until all chunks complete)
```

---

## Code Example

```java
import java.util.*;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class ParallelStreamsDemo {

    public static void main(String[] args) {

        List<Integer> numbers = IntStream.rangeClosed(1, 20_000_000).boxed().collect(Collectors.toList());

        // Sequential vs parallel timing for a genuinely expensive operation
        long start = System.currentTimeMillis();
        long sequentialSum = numbers.stream().mapToLong(n -> expensiveComputation(n)).sum();
        long sequentialTime = System.currentTimeMillis() - start;

        start = System.currentTimeMillis();
        long parallelSum = numbers.parallelStream().mapToLong(n -> expensiveComputation(n)).sum();
        long parallelTime = System.currentTimeMillis() - start;

        System.out.println("Sequential: " + sequentialSum + " in " + sequentialTime + "ms");
        System.out.println("Parallel:   " + parallelSum + " in " + parallelTime + "ms");

        // DANGEROUS pattern: shared mutable state
        List<Integer> unsafeResults = new ArrayList<>();
        numbers.subList(0, 100_000).parallelStream().forEach(n -> unsafeResults.add(n)); // RACE CONDITION
        System.out.println("Unsafe collected size: " + unsafeResults.size() + " (expected 100000, likely LESS due to race)");

        // SAFE pattern: proper collect()
        List<Integer> safeResults = numbers.subList(0, 100_000).parallelStream().collect(Collectors.toList());
        System.out.println("Safe collected size: " + safeResults.size() + " (should be EXACTLY 100000)");
    }

    static long expensiveComputation(int n) {
        long result = 0;
        for (int i = 0; i < 10; i++) result += Math.sqrt(n) * i; // artificial CPU work
        return (long) result;
    }
}
```

### Line-by-line explanation

1. The sequential vs. parallel timing comparison uses a genuinely CPU-expensive `expensiveComputation()` per element (not trivial addition) across 20 million elements — a scenario where parallel streams should show a **real, measurable** speedup on a multi-core machine.
2. The "dangerous pattern" directly reproduces the check-then-act-style race condition from [[26 - Race Condition]], but via `ArrayList.add()` inside a parallel `forEach` — the collected size will likely be **less than** 100,000 due to lost updates from concurrent, unsynchronized modification.
3. The "safe pattern" uses `.collect(Collectors.toList())`, which is specifically designed to handle the parallel-combining logic correctly and safely, reliably producing exactly 100,000 elements every run.

---

## Dry Run

```
Sequential: [large number] in 1850ms
Parallel:   [large number] in 480ms
Unsafe collected size: 97834 (expected 100000, likely LESS due to race)
Safe collected size: 100000 (should be EXACTLY 100000)
```

(Exact numbers vary by machine — the core lesson is the directional pattern: parallel wins on genuinely expensive work, `collect()` is safe, manual `forEach` + shared mutable list is not.)

---

## Common Mistakes

1. **Adding `.parallel()` reflexively to "make things faster" without measuring.**
    
    - For small datasets or cheap per-element operations, the overhead of splitting/coordinating work across threads can make parallel streams **slower** than sequential — always measure, don't assume.
2. **Using `forEach()` with a shared, non-thread-safe collection to accumulate results.**
    
    - This is a genuine, common, real-world race condition — use `collect()` instead, which handles the parallel-safe accumulation logic correctly.
3. **Relying on element processing order in a parallel stream.**
    
    - Order is not guaranteed unless you specifically use `forEachOrdered()` (which limits the parallelism benefit) — for anything order-sensitive, be very careful, or reconsider whether parallelism is appropriate at all for that specific operation.
4. **Not considering that parallel streams share the JVM-wide common `ForkJoinPool`.**
    
    - Heavy parallel stream usage in one part of an application can starve or slow down unrelated `CompletableFuture`/parallel stream work happening concurrently elsewhere in the same JVM.

---

## Production Usage

- **Large batch data transformations** (processing large in-memory datasets, e.g., transforming a large list of records) are a legitimate, common use case for parallel streams, when per-element work is genuinely substantial.
- **Most production code uses `.stream()` (sequential), not `.parallelStream()`**, precisely because the majority of everyday collection processing doesn't involve datasets large enough or per-element work expensive enough to benefit — this is worth internalizing as the honest default expectation.
- **Careful, measured adoption**: teams that do use parallel streams in production typically benchmark specific hot paths first, rather than applying it broadly as a default optimization strategy.
- **Avoided entirely in request-handling code paths** in many web applications, specifically because of the shared common pool contention risk — using a request-handling thread pool's own concurrency (many concurrent requests, each processed sequentially) is often a better fit than adding another layer of parallelism within each individual request's processing.

---

## Interview Questions

**Beginner:**

1. What is a parallel stream, and how do you create one?
2. What underlying mechanism does `.parallelStream()` use?
3. Does a parallel stream guarantee faster execution than a sequential stream?

**Intermediate:** 4. Why is using `forEach()` to add to a shared `ArrayList` inside a parallel stream dangerous? 5. What is the safe alternative to accumulating results from a parallel stream? 6. Does a parallel stream guarantee element processing order?

**Advanced:** 7. Explain, step by step, how a parallel stream splits and processes a large `List`, connecting this to `ForkJoinPool`'s work-stealing mechanism. 8. Why can heavy parallel stream usage in one part of an application affect unrelated code elsewhere in the same JVM?

**Production-Level:** 9. What criteria would you use to decide whether a specific data-processing operation is a good candidate for parallel streams? 10. Why do many production codebases default to sequential streams, reserving parallel streams for specifically measured, justified hot paths?

---

## Follow-up Questions

- Can you provide a custom `ForkJoinPool` for a parallel stream, instead of using the common pool? _(Yes, though somewhat awkwardly — by submitting the parallel stream operation as a task to a custom `ForkJoinPool` via `.submit()`, since there's no direct, first-class API for specifying a custom pool on the stream itself.)_
- Is `.parallelStream()` on a `Set` or `Map` different from a `List`? _(The underlying mechanism is the same (Spliterator-based splitting via the common pool), though the actual splitting efficiency can vary by the underlying data structure's characteristics.)_
- Does converting a sequential stream to parallel mid-chain (`.stream().parallel()`) behave differently from starting with `.parallelStream()`? _(No — both result in the same parallel execution mode for the remainder of the stream pipeline; `.parallel()` can even be called on an already-parallel stream, or `.sequential()` to switch back, with the LAST such call in the chain determining the final mode.)_

---

## Best Practices

- Measure before adopting parallel streams — don't assume they're faster without benchmarking your specific workload.
- Never use `forEach()` with a shared, non-thread-safe collection inside a parallel stream — use `collect()` instead.
- Avoid relying on element processing order unless using `forEachOrdered()` (and understand this reduces the parallelism benefit).
- Be mindful of the shared common `ForkJoinPool` — avoid heavy parallel stream usage in latency-sensitive request-handling paths where pool contention could affect unrelated work.

---

## Performance Notes

- Parallel streams genuinely shine for large datasets with computationally expensive, stateless, independent per-element operations — exactly the same conditions that favor `ForkJoinPool` in general (see [[48 - Fork Join Pool]]).
- For small datasets or cheap operations, sequential streams are often faster due to avoided splitting/coordination overhead — this is not a rare edge case; it's the common case for typical, everyday collection sizes in most applications.
- Always benchmark on your actual target hardware and realistic data sizes — parallel stream performance characteristics can vary significantly based on core count and workload shape.

---

## Comparison Table

|Aspect|Sequential Stream|Parallel Stream|
|---|---|---|
|Execution|Single thread|Common `ForkJoinPool`, multiple threads|
|Element order|Guaranteed|Not guaranteed (unless `forEachOrdered`)|
|Best for|Most everyday use cases; small-to-medium datasets|Large datasets, expensive per-element work|
|Risk|None additional|Race conditions if using non-thread-safe shared state|

---

## Summary

- `.parallelStream()` applies fork/join, divide-and-conquer parallelism (via the common `ForkJoinPool`, see [[48 - Fork Join Pool]]) to stream processing, with a single method call.
- It genuinely helps for large datasets with expensive, stateless, independent per-element work; it can hurt (via overhead) for small datasets or cheap operations.
- Using `forEach()` with a shared, non-thread-safe collection is a real, common race condition — use `collect()` instead.
- Element order is not guaranteed in parallel streams unless explicitly preserved via `forEachOrdered()`.
- Parallel streams share the JVM-wide common pool, so heavy usage can affect unrelated concurrent work elsewhere in the application.
- Sequential streams remain the correct default for most everyday use; parallel streams should be a deliberate, measured choice for specific, justified scenarios.

---

## Cheat Sheet

|Rule|Why|
|---|---|
|Measure before using `.parallelStream()`|Overhead can outweigh benefit for small/cheap workloads|
|Never `forEach()` into a shared `ArrayList`|Race condition — use `collect()` instead|
|Don't rely on order|Not guaranteed unless `forEachOrdered()`|
|Watch for common pool contention|Shared JVM-wide with `CompletableFuture` and other parallel streams|

---

## Keywords

Parallel Stream, ForkJoinPool, Spliterator, collect, forEach, Race Condition, Common Pool

---

## Related Notes

[[48 - Fork Join Pool]] [[26 - Race Condition]] [[51 - Concurrent Collections]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Compare sequential vs. parallel stream timing for a simple, cheap operation (like doubling numbers) on a small list, and observe that parallel may not be faster (or could even be slower).
2. Reproduce this note's expensive-computation benchmark and confirm a measurable speedup with parallel streams on a large dataset.

**Medium:** 3. Reproduce the dangerous `forEach()` + shared `ArrayList` race condition, running it multiple times and observing inconsistent (incorrect) collected sizes. 4. Fix it using `collect(Collectors.toList())` and confirm consistent, correct results across many runs.

**Hard:** 5. Design an experiment demonstrating that parallel stream element order is not guaranteed (e.g., print elements via `forEach()` and observe out-of-order output), then fix it with `forEachOrdered()` and observe the performance cost of doing so.

---

## Mini Project

**Project: Parallel Stream Decision Helper**

Build a small console application that:

1. Accepts a dataset size and a "per-element cost" simulation parameter.
2. Runs both sequential and parallel versions of a stream operation with those parameters, timing both.
3. Repeats across a range of dataset sizes (small, medium, large) and per-element costs (cheap, moderate, expensive).
4. Prints a summary table showing at which combinations parallel streams actually provide a speedup, versus where they're neutral or harmful.

**Goal:** Build a data-driven, empirical decision framework for when parallel streams are actually worth using — replacing guesswork with measured evidence, directly addressing the core "should I use `.parallelStream()`?" question with real data.

---

## Common Interview Trap

**Trap Question:** _"Since parallel streams automatically use multiple threads, is it always safe to use them for any stream operation, as long as I don't modify any variables outside the lambda?"_

The trap is assuming "don't modify external variables" is sufficient to guarantee safety, missing the specific, common danger of calling a **mutating method on a shared object** (like `list.add()`) — which isn't "modifying a variable" in the reassignment sense, but **is** modifying shared, mutable state via a method call, and is exactly as dangerous as any other race condition covered in [[26 - Race Condition]]. The correct, more complete answer: parallel streams are safe specifically when each element's processing is **fully independent** — no shared mutable state is read or written by the lambda, whether through variable reassignment OR through method calls on shared objects (like a shared `ArrayList`, shared counter object, shared `Map`, etc.). The safe, idiomatic way to accumulate results from a parallel stream is always through the stream API's own combining mechanisms (`collect()`, `reduce()`), which handle the necessary thread-safety internally — never through manually calling mutating methods on an externally shared object from within a parallel `forEach()`.

---

## Frequently Asked Questions

**Q: Does `.parallelStream()` always use as many threads as CPU cores?** A: By default, the common `ForkJoinPool`'s parallelism level is `availableProcessors() - 1`, but the actual number of threads genuinely used for a specific operation also depends on how the data splits and how much work is actually available to parallelize — it's not a fixed, guaranteed thread count for every operation.

**Q: Can you convert a parallel stream back to sequential mid-pipeline?** A: Yes — calling `.sequential()` anywhere in the chain switches subsequent processing to sequential mode; as noted, the LAST such call (`.parallel()` or `.sequential()`) in the pipeline determines the final execution mode.

**Q: Is there a performance cost to simply calling `.parallelStream()` on a stream that turns out to have very little work to do?** A: Yes — there's inherent overhead in the splitting/task-submission machinery, even if ultimately little "real" parallel work happens; this is precisely why small datasets or cheap operations can see parallel streams underperform sequential ones.

**Q: Do all terminal operations combine results the same way?** A: No — different terminal operations have different combining strategies internally (e.g., `sum()` uses simple numeric addition, `collect(Collectors.toList())` has its own thread-safe accumulation strategy, custom `reduce()` calls use whatever combining function you provide) — this is why using the stream API's own terminal operations, rather than manual side-effecting `forEach()`, is the safe, correct approach.

---

## Revision Checklist

- [ ] I understand that parallel streams use the common `ForkJoinPool` internally.
- [ ] I can identify when parallel streams genuinely help versus when they hurt.
- [ ] I understand exactly why `forEach()` + shared `ArrayList` is a race condition, and know the safe alternative.
- [ ] I understand that element order is not guaranteed in parallel streams by default.
- [ ] I understand the shared common pool contention risk across unrelated parallel stream/CompletableFuture usage.

---

## Flashcards

**Q:** What underlying mechanism does `.parallelStream()` use by default? **A:** The common `ForkJoinPool`.

**Q:** Why is `numbers.parallelStream().forEach(n -> sharedList.add(n))` dangerous? **A:** `ArrayList.add()` is not thread-safe — concurrent calls from multiple threads can cause a race condition, corrupting the list or losing elements.

**Q:** What's the safe way to accumulate results from a parallel stream? **A:** Use `.collect(Collectors.toList())` (or another appropriate collector), not manual `forEach()` into a shared collection.

**Q:** Does a parallel stream guarantee element processing order? **A:** No — unless you use `forEachOrdered()`, which also reduces the parallelism benefit.

**Q:** When do parallel streams genuinely provide a performance benefit? **A:** Large datasets with computationally expensive, stateless, independent per-element operations.

---

## References

- Oracle Java Tutorials — Aggregate Operations (Parallelism): https://docs.oracle.com/javase/tutorial/collections/streams/parallelism.html
- `java.util.stream.Stream` API Documentation (`parallel`, `sequential`)
- `java.util.concurrent.ForkJoinPool` API Documentation (common pool)