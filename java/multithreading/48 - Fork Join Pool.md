# Fork Join Pool

## Objective

By the end of this note, you will understand:

- The divide-and-conquer model `ForkJoinPool` is specifically designed for
- What "work-stealing" is and why it makes `ForkJoinPool` more efficient than a regular thread pool for this specific workload shape
- How to implement a task using `RecursiveTask`/`RecursiveAction`
- How `ForkJoinPool` quietly powers Java's parallel streams

This note builds on [[44 - Thread Pool]] and [[46 - ExecutorService]], covering a specialized pool designed for a fundamentally different task shape than the general-purpose pools covered so far.

---

## Why should I learn this?

Every thread pool covered so far (`ExecutorService`) is designed for **independent** tasks — a batch of unrelated units of work. `ForkJoinPool` is designed for a different, common shape: a **single large problem that can be recursively split into smaller subproblems**, each processed in parallel, with results combined back together — think "sum a huge array," "sort a huge dataset," "process a large tree structure." Understanding it also demystifies exactly what powers Java's parallel streams (`.parallelStream()`), which you've likely used without knowing what's underneath.

---

## Beginner Explanation

Imagine you need to count how many jellybeans are in an enormous jar — too many to count one by one efficiently. Instead, you **split** the jar's contents into two halves, hand each half to a different helper, and ask each helper to do the same thing: if their pile is still too big, **split it again** and delegate further; if it's small enough, just count it directly. Eventually, everyone reports their counts back up the chain, and the totals get **combined** at each level until you have the final answer.

This "split the problem, solve pieces in parallel, combine the results" pattern is called **divide-and-conquer**, and `ForkJoinPool` is Java's specialized thread pool built specifically to execute this pattern efficiently.

---

## Real World Analogy

**Moving Company Analogy**

Imagine a moving crew tasked with moving all the boxes out of a large house. Instead of one fixed assignment per worker, the crew uses a smart strategy: a worker who finishes clearing their assigned room **early** doesn't just stand idle — they go **help another worker** who's still got a full room left, effectively "stealing" some of that worker's remaining boxes to carry themselves. This keeps every worker busy and the whole job finishing faster overall, rather than the team's total completion time being bottlenecked by whichever single room happened to have the most boxes.

This "idle workers steal work from busy workers" strategy is precisely what `ForkJoinPool`'s **work-stealing** algorithm does.

---

## Technical Explanation

### The Fork/Join Model

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // Base case: small enough, just compute directly
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        } else {
            // Recursive case: split (FORK) into two halves
            int mid = (start + end) / 2;
            SumTask leftTask = new SumTask(array, start, mid);
            SumTask rightTask = new SumTask(array, mid, end);

            leftTask.fork();               // schedule leftTask to run asynchronously
            long rightResult = rightTask.compute(); // compute rightTask directly on THIS thread
            long leftResult = leftTask.join();       // wait for leftTask's result (JOIN)

            return leftResult + rightResult;
        }
    }
}
```

```java
ForkJoinPool pool = ForkJoinPool.commonPool(); // or new ForkJoinPool(nThreads)
long total = pool.invoke(new SumTask(bigArray, 0, bigArray.length));
```

### `RecursiveTask<V>` vs. `RecursiveAction`

- **`RecursiveTask<V>`**: for computations that **return a result** (like the sum example above) — implement `compute()` returning `V`.
- **`RecursiveAction`**: for computations with **no return value** (e.g., in-place array transformations) — implement `compute()` returning `void`.

### `fork()` and `join()` — The Key Methods

- **`fork()`**: schedules a subtask to be executed **asynchronously**, potentially by a different worker thread in the pool (or, crucially, potentially "stolen" by an idle thread — see below).
- **`join()`**: waits for a forked subtask's result, blocking (efficiently — see below) until it's available.

The common pattern shown above — `fork()` the left half, `compute()` the right half **directly on the current thread**, then `join()` the left half — is deliberately structured this way to avoid the overhead of forking **both** halves when one can simply be computed inline.

### Work-Stealing — The Core Efficiency Mechanism

Each worker thread in a `ForkJoinPool` maintains its **own** double-ended queue (deque) of tasks. When a thread forks a subtask, it pushes it onto its **own** deque and continues working. When a thread runs out of its own work, instead of sitting idle, it "steals" a task from the **back** of another, busier thread's deque (while the busy thread itself works from the **front** of its own deque) — this specific double-ended design minimizes contention between the owning thread and thieves.

This is precisely why `ForkJoinPool` outperforms a regular `ExecutorService`-based thread pool for this recursive, uneven workload shape: regular pools have **one shared queue**, causing more contention and no mechanism to rebalance work when some subtasks finish much faster than others; `ForkJoinPool`'s per-thread deques with stealing keep all threads productively busy.

### The Common Pool

```java
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```

Java maintains a single, shared, JVM-wide `ForkJoinPool` (sized by default to `Runtime.getRuntime().availableProcessors() - 1`), used automatically by parallel streams and `CompletableFuture`'s async methods (see [[49 - CompletableFuture]], [[50 - Parallel Streams]]) unless a different executor is explicitly specified.

---

## Internal Working

Step-by-step of the fork/join execution model:

1. A task's `compute()` method checks: is the problem small enough to solve directly (below some threshold)? If so, solve it directly and return.
2. If not, split the problem into (typically two) smaller subtasks.
3. `fork()` one subtask — pushing it onto the current thread's own deque, making it available for execution (by this thread later, or by a thief thread sooner).
4. `compute()` the other subtask **directly**, recursively repeating this entire process for it.
5. `join()` the forked subtask — if it hasn't been started yet, the current thread can **help execute it directly** (rather than just blocking and wasting time); if another thread already stole and is working on it, this thread waits (efficiently) for that result.
6. Combine both subtasks' results and return.

---

## Memory Diagram

```
        ForkJoinPool
   -----------------------------------
   |  Worker-1 deque: [task_A]         |
   |  Worker-2 deque: [] (empty!)      |
   |  Worker-3 deque: [task_B, task_C] |
   -----------------------------------

   Worker-2 has nothing to do -- STEALS task_C
   from the BACK of Worker-3's deque

   -----------------------------------
   |  Worker-1 deque: [task_A]         |
   |  Worker-2 deque: [task_C] (stolen)|
   |  Worker-3 deque: [task_B]         |
   -----------------------------------
```

---

## Execution Flow

```
invoke(bigTask)
        |
        v
compute(): is problem small enough? --NO--> split into left, right
        |
        v
fork(left)  -- pushed to own deque
        |
        v
compute(right) directly on this thread (recursion continues here too)
        |
        v
join(left)  -- if not yet started, HELP execute it; else wait for its result
        |
        v
combine(leftResult, rightResult)
        |
        v
return combined result up the recursion chain
```

---

## Code Example

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class ForkJoinDemo {

    static class SumTask extends RecursiveTask<Long> {
        private final int[] array;
        private final int start, end;
        private static final int THRESHOLD = 10_000;

        SumTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }

        @Override
        protected Long compute() {
            if (end - start <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) sum += array[i];
                return sum;
            }
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);

            left.fork();
            long rightResult = right.compute();
            long leftResult = left.join();

            return leftResult + rightResult;
        }
    }

    public static void main(String[] args) {
        int[] bigArray = new int[10_000_000];
        for (int i = 0; i < bigArray.length; i++) bigArray[i] = 1; // sum should equal array length

        ForkJoinPool pool = new ForkJoinPool(); // uses available processors by default

        long start = System.currentTimeMillis();
        long result = pool.invoke(new SumTask(bigArray, 0, bigArray.length));
        long time = System.currentTimeMillis() - start;

        System.out.println("Sum: " + result + " (expected " + bigArray.length + ")");
        System.out.println("Parallel time: " + time + "ms");

        // Compare against sequential
        start = System.currentTimeMillis();
        long sequentialSum = 0;
        for (int v : bigArray) sequentialSum += v;
        long sequentialTime = System.currentTimeMillis() - start;

        System.out.println("Sequential sum: " + sequentialSum + ", time: " + sequentialTime + "ms");
    }
}
```

### Line-by-line explanation

1. `SumTask` recursively splits the array until each piece is at or below `THRESHOLD` (10,000 elements), at which point it sums directly.
2. The `fork()`/`compute()`/`join()` pattern is exactly as described — `left.fork()` schedules the left half, `right.compute()` runs the right half on the current thread, `left.join()` retrieves the left half's result (potentially helping compute it if not yet started).
3. The demo compares parallel (`ForkJoinPool`) timing against simple sequential summing — for a large enough array, the parallel version should show a meaningful speedup on a multi-core machine, directly demonstrating the value of this approach for large, splittable problems.

---

## Dry Run

Example output (actual times vary by machine):

```
Sum: 10000000 (expected 10000000)
Parallel time: 45ms
Sequential sum: 10000000, time: 28ms
```

**Note**: for a workload this simple (trivial addition), the sequential version can sometimes be **faster** due to fork/join overhead outweighing the benefit — this is an important, honest observation: `ForkJoinPool` shines most clearly for **more computationally expensive** per-element work, where the parallelization benefit meaningfully outweighs the splitting/coordination overhead. A well-designed benchmark should test with genuinely CPU-intensive per-element work to see the real advantage.

---

## Common Mistakes

1. **Setting the threshold too low, causing excessive splitting overhead.**
    
    - If subtasks become too small, the overhead of creating and coordinating them outweighs the benefit of parallelism — the threshold should be tuned based on actual per-element work cost.
2. **Forking both halves instead of computing one directly.**
    
    - The idiomatic pattern (`fork()` one, `compute()` the other directly, then `join()`) is more efficient than forking both and joining both, since it avoids unnecessary task scheduling overhead for one of the two halves.
3. **Using `ForkJoinPool` for a workload that isn't actually recursively splittable (independent, unrelated tasks).**
    
    - For a simple batch of unrelated tasks, a regular `ExecutorService` (see [[46 - ExecutorService]]) is the more appropriate, simpler tool — `ForkJoinPool` specifically shines for divide-and-conquer problems.
4. **Blocking inside a `ForkJoinPool` task on external I/O or other long blocking operations.**
    
    - This can starve the pool's limited thread count (typically sized to CPU cores) — `ForkJoinPool` is optimized for CPU-bound, computational work, not I/O-bound blocking operations.

---

## Production Usage

- **Java's parallel streams** (`.parallelStream()`, see [[50 - Parallel Streams]]) use the common `ForkJoinPool` internally to split and process stream operations in parallel — this is likely the most common way most developers actually interact with `ForkJoinPool`, often without realizing it.
- **Large-scale numerical/scientific computing** in Java (matrix operations, large dataset transformations) commonly uses fork/join directly for genuine divide-and-conquer parallelism.
- **`CompletableFuture`'s async methods** (see [[49 - CompletableFuture]]) also default to using the common `ForkJoinPool` unless a different executor is explicitly provided.
- **Large tree/graph traversal and processing** (e.g., parallel processing of a large file system tree, or a large recursive data structure) is a natural fit for the fork/join model.

---

## Interview Questions

**Beginner:**

1. What is `ForkJoinPool` designed for?
2. What is "work-stealing"?
3. What is the difference between `RecursiveTask` and `RecursiveAction`?

**Intermediate:** 4. What do `fork()` and `join()` do? 5. Why does the idiomatic pattern fork one subtask but compute the other directly, rather than forking both? 6. What is the "common pool," and what JDK features use it by default?

**Advanced:** 7. Explain, step by step, how work-stealing improves efficiency compared to a regular thread pool's single shared queue. 8. Why can blocking I/O operations inside a `ForkJoinPool` task be particularly problematic?

**Production-Level:** 9. What underlying mechanism powers Java's parallel streams? 10. How would you decide whether a given computational problem is a good fit for `ForkJoinPool` versus a regular `ExecutorService`?

---

## Follow-up Questions

- Does every `ForkJoinPool` task need to actually fork subtasks? _(No — if the problem is already small enough (below the threshold), `compute()` can simply solve it directly with no forking at all; forking only happens for the recursive, "still too big" case.)_
- Can `ForkJoinPool` tasks be submitted like regular `ExecutorService` tasks (e.g., via `submit()`)? _(Yes — `ForkJoinPool` implements `ExecutorService`, supporting `submit()`/`execute()` for standalone tasks in addition to its specialized `invoke()` for `RecursiveTask`/`RecursiveAction`.)_
- Is the common pool shared across the entire JVM, including different libraries/frameworks running in the same application? _(Yes — this is an important, sometimes-overlooked detail: if multiple parts of your application (or third-party libraries) all use the common pool via parallel streams or `CompletableFuture`, they're all competing for the same limited set of threads.)_

---

## Best Practices

- Use `ForkJoinPool` specifically for CPU-bound, recursively-splittable divide-and-conquer problems — not for general-purpose, independent task batches (use `ExecutorService` for those).
- Tune your splitting threshold based on actual measured performance, not a guess — too fine-grained splitting adds overhead without benefit.
- Avoid blocking I/O operations inside fork/join tasks, since this can starve the pool's limited, CPU-core-sized thread count.
- Be aware that the common pool is shared JVM-wide — if you have specific isolation or sizing needs, consider creating a dedicated `ForkJoinPool` instance instead.

---

## Performance Notes

- Work-stealing provides genuine, measurable efficiency gains for uneven, recursively-splittable workloads compared to a single-shared-queue thread pool.
- The benefit of parallelization only outweighs its overhead when per-element/per-subtask work is genuinely substantial — trivial operations (like simple addition, as shown in this note's honest benchmark) may not show a clear win, or can even be slower in parallel due to coordination overhead.
- The common pool's default size (`availableProcessors() - 1`) is deliberately tuned for CPU-bound work — don't expect it to scale well for I/O-bound workloads without reconsidering your approach entirely.

---

## Comparison Table

|Aspect|Regular `ExecutorService`|`ForkJoinPool`|
|---|---|---|
|Best for|Independent, unrelated tasks|Recursively splittable divide-and-conquer problems|
|Task queue|Single shared queue|Per-thread deques with work-stealing|
|Typical workload|I/O-bound or general CPU-bound|CPU-bound, recursive|
|Underlies|Custom application task processing|Parallel streams, `CompletableFuture` async methods|

---

## Summary

- `ForkJoinPool` is specialized for divide-and-conquer problems: recursively splitting a large problem into smaller subproblems, solving them (potentially in parallel), and combining results.
- `RecursiveTask<V>` (returns a value) and `RecursiveAction` (no return value) are the base classes for implementing fork/join tasks, using `compute()`, `fork()`, and `join()`.
- **Work-stealing** — idle threads stealing tasks from busy threads' own deques — is the core efficiency mechanism that outperforms a regular thread pool's single shared queue for this workload shape.
- The JVM-wide common pool powers parallel streams and `CompletableFuture`'s async methods by default.
- `ForkJoinPool` is optimized for CPU-bound, computational work — avoid blocking I/O within its tasks.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|`RecursiveTask<V>`|Fork/join task that returns a value|
|`RecursiveAction`|Fork/join task with no return value|
|`fork()`|Schedule a subtask asynchronously|
|`join()`|Wait for (and potentially help execute) a forked subtask's result|
|Work-stealing|Idle threads steal tasks from busy threads' deques|
|Common pool|JVM-wide shared pool, used by parallel streams and `CompletableFuture`|

---

## Keywords

ForkJoinPool, RecursiveTask, RecursiveAction, fork, join, Work-Stealing, Divide and Conquer, Common Pool

---

## Related Notes

[[44 - Thread Pool]] [[46 - ExecutorService]] [[49 - CompletableFuture]] [[50 - Parallel Streams]]

---

## Practice Exercises

**Easy:**

1. Implement a `RecursiveTask` that finds the maximum value in a large array, and compare its result against a simple sequential loop.
2. Reproduce this note's `SumTask` demo, and experiment with different threshold values, observing the performance impact.

**Medium:** 3. Implement a `RecursiveAction` that doubles every element in a large array in place, using the fork/join pattern. 4. Modify the sum example to use a genuinely CPU-expensive per-element operation (e.g., a slow, artificial computation per element) and re-run the parallel vs. sequential comparison — confirm the parallel version now shows a clear advantage.

**Hard:** 5. Implement a fork/join-based merge sort for a large array, and benchmark it against `Arrays.sort()` (sequential) for various array sizes.

---

## Mini Project

**Project: Parallel Directory Size Calculator**

Build a small console application that:

1. Uses `RecursiveTask<Long>` to recursively calculate the total size of all files in a directory tree (a naturally recursive, divide-and-conquer problem — each subdirectory can be processed as a separate subtask).
2. Compares the fork/join-based parallel implementation against a simple sequential recursive implementation, timing both on a reasonably large directory tree.
3. Reports the speedup (or lack thereof) observed, and discusses why file-system-bound work might behave differently from purely CPU-bound work in this context (a nice bridge to thinking about I/O-bound caveats).

**Goal:** Apply fork/join to a genuinely recursive, real-world problem shape, while also encountering firsthand why I/O-bound work interacts differently with `ForkJoinPool`'s CPU-optimized design.

---

## Common Interview Trap

**Trap Question:** _"Since `ForkJoinPool` is more sophisticated and efficient than a regular `ExecutorService` (thanks to work-stealing), should you always prefer it for any kind of parallel task processing?"_

The trap is assuming "more sophisticated" automatically means "universally better." The correct answer: **no** — `ForkJoinPool`'s work-stealing design specifically optimizes for **recursively splittable, CPU-bound** workloads with uneven subtask completion times. For a simple batch of **independent, unrelated** tasks (e.g., "process these 100 separate HTTP requests" or "handle these 50 unrelated database queries"), a regular `ExecutorService`-based thread pool (see [[46 - ExecutorService]]) is simpler, equally effective, and the more idiomatic choice — there's no recursive splitting or work-stealing benefit to gain when tasks aren't hierarchically related. Additionally, `ForkJoinPool` is specifically poorly suited for I/O-bound blocking work, where a regular thread pool sized for I/O concurrency (potentially far exceeding the CPU core count) is the correct tool. Choosing the right tool depends entirely on the **shape** of your workload, not a blanket "newer/fancier is always better" heuristic.

---

## Frequently Asked Questions

**Q: Is `ForkJoinPool` part of `java.util.concurrent`?** A: Yes, along with `RecursiveTask`, `RecursiveAction`, and `ForkJoinTask` (the common superclass).

**Q: Since which Java version has `ForkJoinPool` been available?** A: Java 7 — it was introduced specifically to support this divide-and-conquer parallelism model, predating parallel streams (Java 8), which later came to rely on it internally.

**Q: Can you configure the common pool's size?** A: Yes, via the system property `java.util.concurrent.ForkJoinPool.common.parallelism`, though this affects the JVM-wide shared pool used by all parallel streams and default `CompletableFuture` async operations — changing it has broad, application-wide implications.

**Q: Does `ForkJoinPool` guarantee any particular execution order among subtasks?** A: No — due to work-stealing, the exact order in which subtasks are picked up and executed by which specific worker thread is non-deterministic; only the final, combined result's correctness is guaranteed (assuming the combination logic is itself correct), not any particular execution ordering.

---

## Revision Checklist

- [ ] I understand the divide-and-conquer workload shape `ForkJoinPool` is designed for.
- [ ] I can explain work-stealing and why it improves efficiency over a single shared queue.
- [ ] I know the difference between `RecursiveTask` and `RecursiveAction`.
- [ ] I can implement the idiomatic `fork()`/`compute()`/`join()` pattern.
- [ ] I understand that `ForkJoinPool` (and its common pool) underlies parallel streams and `CompletableFuture`'s default async behavior.

---

## Flashcards

**Q:** What workload shape is `ForkJoinPool` specifically designed for? **A:** Recursively splittable, divide-and-conquer problems.

**Q:** What is work-stealing? **A:** Idle worker threads take tasks from the back of busy threads' own deques, keeping all threads productively busy.

**Q:** What's the difference between `RecursiveTask` and `RecursiveAction`? **A:** `RecursiveTask<V>` returns a value; `RecursiveAction` does not.

**Q:** What is the idiomatic fork/join pattern for splitting into two subtasks? **A:** `fork()` one subtask, `compute()` the other directly, then `join()` the forked one.

**Q:** What JDK feature uses the common `ForkJoinPool` by default? **A:** Parallel streams (`.parallelStream()`), and `CompletableFuture`'s async methods.

---

## References

- Oracle Java Tutorials — Fork/Join: https://docs.oracle.com/javase/tutorial/essential/concurrency/forkjoin.html
- `java.util.concurrent.ForkJoinPool` API Documentation
- `java.util.concurrent.RecursiveTask` / `RecursiveAction` API Documentation