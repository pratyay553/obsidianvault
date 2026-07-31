# ExecutorService

## Objective

By the end of this note, you will understand:

- The full `ExecutorService` API surface: submission, batch operations, and shutdown
- Every `Executors` factory method and precisely when to use (or avoid) each
- The proper shutdown sequence and why getting it wrong causes real production issues
- How to configure a custom `ThreadPoolExecutor` directly for full production control

This is the most practically important note in this entire vault — `ExecutorService` is the API you will use constantly in real Java development.

---

## Why should I learn this?

Every concept from Sections 1-8 converges here. `ExecutorService` is how virtually all production Java code actually runs concurrent work — you now understand exactly what happens underneath it (raw threads, thread pools, `BlockingQueue`, `Future`), and this note gives you the complete, practical API mastery to use it correctly and configure it deliberately, rather than just copy-pasting `Executors.newFixedThreadPool(10)` without understanding the implications.

---

## Beginner Explanation

`ExecutorService` is the interface you actually **use** to submit and manage concurrent tasks in real Java code. You obtain one (usually via a factory method on the `Executors` class, or by constructing a `ThreadPoolExecutor` directly), submit tasks to it, optionally retrieve results, and — critically — **shut it down** when you're done, so its threads don't keep your application running forever.

---

## Real World Analogy

**Call Center Analogy**

Think of `ExecutorService` as a call center's management system. You (the application) submit "calls to make" (tasks) to the system. The system has a team of agents (worker threads) who pick up and handle calls as they come in. You can ask for a report on how a specific call went (a `Future`), submit a whole batch of calls at once and wait for all of them (`invokeAll()`), and — at the end of the day — tell the system to stop accepting new calls and wind down (`shutdown()`), letting agents finish calls already in progress but not start new ones.

---

## Technical Explanation

### The `Executors` Factory Methods

```java
ExecutorService fixed = Executors.newFixedThreadPool(4);
```

A fixed-size pool of exactly 4 threads. If all are busy, tasks queue on an **unbounded** `LinkedBlockingQueue` internally. Good default for CPU-bound work with a known, stable thread count need.

```java
ExecutorService cached = Executors.newCachedThreadPool();
```

Creates new threads as needed (no fixed upper bound), reuses idle threads, and terminates threads idle for 60 seconds. Uses a `SynchronousQueue` internally (zero buffering — see [[43 - BlockingQueue]]), meaning tasks are handed directly to a thread (existing or newly created) rather than queued. Good for many short-lived, I/O-bound tasks, but **dangerous** under unexpected load spikes since it can create an unbounded number of threads.

```java
ExecutorService single = Executors.newSingleThreadExecutor();
```

Exactly one worker thread, processing tasks strictly in order — useful when you need guaranteed sequential execution of otherwise-async submitted tasks.

```java
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
```

Supports delayed/periodic execution (full detail in [[47 - ScheduledExecutorService]]).

**Important production caveat**: `Executors`' convenience factory methods use **unbounded queues** (fixed, single-thread) or **unbounded thread creation** (cached) by default — both are real production risk factors under sustained overload, which is exactly why constructing `ThreadPoolExecutor` directly (shown below) is often recommended for production systems needing explicit resource bounds.

### Submitting Tasks: `execute()` vs. `submit()`

```java
executorService.execute(runnable);          // fire-and-forget, no Future, exceptions go to uncaught handler
Future<?> f1 = executorService.submit(runnable);      // returns a Future<?> (result is always null on success)
Future<Integer> f2 = executorService.submit(callable); // returns a Future<Integer> with the actual result
```

`execute()` (inherited from the base `Executor` interface) provides no way to observe completion or catch exceptions programmatically. `submit()` (added by `ExecutorService`) always returns a `Future`, letting you retrieve results and — critically — **exceptions**, which are otherwise silently swallowed inside `execute()`'s fire-and-forget model (a common, costly production mistake: using `execute()` for something whose failures actually need to be noticed).

### Batch Operations: `invokeAll()` and `invokeAny()`

```java
List<Callable<Integer>> tasks = List.of(task1, task2, task3);

List<Future<Integer>> results = executorService.invokeAll(tasks); // waits for ALL to complete
Integer firstResult = executorService.invokeAny(tasks);            // waits for the FIRST to complete successfully, cancels the rest
```

`invokeAll()` submits all tasks and blocks until **every** one completes (successfully or with an exception, captured in each respective `Future`). `invokeAny()` submits all tasks but returns as soon as **any one** succeeds, cancelling the remaining still-running tasks — useful for scenarios like querying multiple redundant data sources and taking whichever responds first.

### The Shutdown Lifecycle — Critical, Frequently Misunderstood

```java
executorService.shutdown();       // graceful: stop accepting NEW tasks, let queued/running tasks finish
executorService.shutdownNow();    // aggressive: attempt to stop everything NOW, interrupt running tasks, return unstarted tasks
boolean terminated = executorService.awaitTermination(30, TimeUnit.SECONDS); // wait up to 30s for full shutdown
```

**The standard, recommended shutdown sequence**:

```java
executorService.shutdown();
try {
    if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
        executorService.shutdownNow(); // force it if graceful shutdown takes too long
        if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
            System.err.println("Pool did not terminate cleanly.");
        }
    }
} catch (InterruptedException e) {
    executorService.shutdownNow();
    Thread.currentThread().interrupt();
}
```

Recall from [[13 - Daemon Thread]]/[[14 - User Thread]]: `ExecutorService` threads are **non-daemon by default** — forgetting to call `shutdown()` at all is precisely the classic "why won't my JVM exit?" bug.

### Configuring `ThreadPoolExecutor` Directly (Production-Grade Control)

```java
ExecutorService customPool = new ThreadPoolExecutor(
    4,                                  // core pool size
    8,                                  // maximum pool size
    60L, TimeUnit.SECONDS,              // keep-alive time for extra threads
    new ArrayBlockingQueue<>(100),      // BOUNDED task queue -- avoids unbounded memory growth
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy when both pool AND queue are full
);
```

This gives full, explicit control over exactly the parameters `Executors`' convenience methods hide — bounded queue capacity and a deliberate rejection policy are especially important production considerations.

### Rejection Policies (What Happens When the Pool is Overwhelmed)

|Policy|Behavior|
|---|---|
|`AbortPolicy` (default)|Throws `RejectedExecutionException`|
|`CallerRunsPolicy`|The submitting thread itself runs the task (natural backpressure/throttling)|
|`DiscardPolicy`|Silently drops the task|
|`DiscardOldestPolicy`|Drops the oldest queued task, then retries submission|

---

## Internal Working

Step-by-step of `ThreadPoolExecutor`'s task-handling decision logic (this is the actual algorithm):

1. If fewer than **core pool size** threads are running, create a new thread to run the task, even if other threads are idle.
2. Otherwise, if the task queue has room, add the task to the queue (existing threads will pick it up as they free up).
3. Otherwise, if fewer than **maximum pool size** threads are running, create a new (temporary) thread to run the task.
4. Otherwise, the task is **rejected**, per the configured `RejectedExecutionHandler` policy.

This precise order (core threads first, then queue, then extra threads up to max, then reject) is exactly why `Executors.newFixedThreadPool()` (core == max, unbounded queue) never actually creates "extra" threads beyond its fixed size — tasks simply queue indefinitely instead once all fixed threads are busy.

---

## Memory Diagram

```
   New task arrives
        |
        v
   Threads running < core size? --YES--> create new thread, run task
        | NO
        v
   Queue has room? --YES--> enqueue task
        | NO
        v
   Threads running < max size? --YES--> create new (temp) thread, run task
        | NO
        v
   REJECT (per RejectedExecutionHandler policy)
```

---

## Execution Flow

```
Application code: executorService.submit(task)
        |
        v
ThreadPoolExecutor's internal decision logic (see above)
        |
        v
Task eventually runs on a worker thread
        |
        v
Future is populated with result/exception
        |
        v
Application calls future.get() to retrieve it (see [[07 - Future]])
        |
   (eventually)
        |
        v
Application calls shutdown() + awaitTermination()
        |
        v
Pool cleanly stops all threads
```

---

## Code Example

```java
import java.util.concurrent.*;
import java.util.List;
import java.util.ArrayList;

public class ExecutorServiceDemo {

    public static void main(String[] args) throws Exception {

        // Custom, production-style configuration
        ExecutorService pool = new ThreadPoolExecutor(
                2, 4, 30, TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(5),
                new ThreadPoolExecutor.CallerRunsPolicy()
        );

        // submit() with Future
        Future<Integer> future = pool.submit(() -> {
            Thread.sleep(100);
            return 10 * 10;
        });
        System.out.println("submit() result: " + future.get());

        // invokeAll()
        List<Callable<Integer>> tasks = new ArrayList<>();
        for (int i = 1; i <= 5; i++) {
            int n = i;
            tasks.add(() -> n * n);
        }
        List<Future<Integer>> results = pool.invokeAll(tasks);
        for (Future<Integer> f : results) {
            System.out.println("invokeAll() result: " + f.get());
        }

        // invokeAny()
        List<Callable<String>> raceTasks = List.of(
                () -> { Thread.sleep(300); return "Slow service"; },
                () -> { Thread.sleep(50); return "Fast service"; }
        );
        String winner = pool.invokeAny(raceTasks);
        System.out.println("invokeAny() winner: " + winner);

        // Proper shutdown sequence
        pool.shutdown();
        if (!pool.awaitTermination(10, TimeUnit.SECONDS)) {
            pool.shutdownNow();
        }
        System.out.println("Pool terminated: " + pool.isTerminated());
    }
}
```

### Line-by-line explanation

1. The custom `ThreadPoolExecutor` is configured with core=2, max=4, a bounded queue of 5, and `CallerRunsPolicy` for graceful overload handling.
2. `submit()` demonstrates basic result retrieval.
3. `invokeAll()` submits 5 tasks and waits for all to complete, returning their results in the same order as submitted.
4. `invokeAny()` returns as soon as the faster task ("Fast service," 50ms) completes, without waiting for the slower one (300ms) — that slower task is cancelled internally.
5. The shutdown sequence follows the recommended pattern: graceful `shutdown()`, bounded `awaitTermination()`, falling back to `shutdownNow()` if needed.

---

## Dry Run

```
submit() result: 100
invokeAll() result: 1
invokeAll() result: 4
invokeAll() result: 9
invokeAll() result: 16
invokeAll() result: 25
invokeAny() winner: Fast service
Pool terminated: true
```

---

## Common Mistakes

1. **Using `execute()` for tasks whose exceptions actually matter.**
    
    - Exceptions thrown inside a task submitted via `execute()` go to the thread's uncaught exception handler (typically just printed, easily missed in production logs) — use `submit()` and check the returned `Future` if you need to reliably detect failures.
2. **Never calling `shutdown()`.**
    
    - As emphasized throughout Section 3, this is the classic "why won't my JVM exit?" bug, since pool threads are non-daemon by default.
3. **Using `Executors.newCachedThreadPool()` or `newFixedThreadPool()` in production without considering their unbounded queue/thread-creation behavior.**
    
    - Both can lead to unbounded resource consumption under unexpected load — constructing `ThreadPoolExecutor` directly, with an explicit bounded queue and rejection policy, is the recommended production practice for services needing predictable resource limits.
4. **Calling `shutdownNow()` as the default/first shutdown call, when a graceful `shutdown()` would be more appropriate.**
    
    - `shutdownNow()` interrupts running tasks immediately, which can leave work half-done if tasks aren't designed to handle interruption gracefully (see [[21 - Interrupt()]]) — reserve it as a fallback after a bounded `awaitTermination()` following a graceful `shutdown()`.

---

## Production Usage

- **Nearly every Spring Boot application** uses `ExecutorService` (often wrapped in Spring's `TaskExecutor` abstraction) for `@Async` methods, background jobs, and more.
- **HTTP client libraries** (like those used for calling downstream microservices) often use dedicated `ExecutorService` instances to manage connection/request concurrency independently from the main application thread pool.
- **Batch processing systems** use `invokeAll()`-style patterns to process large datasets in parallel chunks, waiting for the entire batch to complete before proceeding to the next stage.
- **Production readiness checklists** at serious engineering organizations almost universally include "are all thread pools explicitly bounded, and do they have an appropriate rejection policy?" as a standard review item.

---

## Interview Questions

**Beginner:**

1. What is the difference between `execute()` and `submit()`?
2. What does `shutdown()` do, and how does it differ from `shutdownNow()`?
3. Name three `Executors` factory methods and what they create.

**Intermediate:** 4. What is the difference between `invokeAll()` and `invokeAny()`? 5. What internal queue type does `Executors.newCachedThreadPool()` use, and why does this matter? 6. What is a rejection policy, and when does it get invoked?

**Advanced:** 7. Walk through `ThreadPoolExecutor`'s exact internal decision logic for handling a newly submitted task (core threads, then queue, then max threads, then reject). 8. Why is `Executors.newFixedThreadPool()` considered risky for production use without further configuration, despite being a very commonly used factory method?

**Production-Level:** 9. Describe the recommended, complete shutdown sequence for a production `ExecutorService`, and explain why each step matters. 10. Why would you construct a `ThreadPoolExecutor` directly instead of using an `Executors` factory method in a production system?

---

## Follow-up Questions

- Does calling `shutdown()` immediately stop all worker threads? _(No — it stops the pool from accepting NEW tasks, but already-queued and currently-running tasks are allowed to complete normally.)_
- What happens if you call `submit()` after `shutdown()` has been called? _(It throws `RejectedExecutionException` — the pool is no longer accepting new work.)_
- Can you reuse an `ExecutorService` after it has been fully shut down? _(No — once terminated, a new `ExecutorService` instance must be created; there's no "restart" capability.)_

---

## Best Practices

- Prefer `submit()` over `execute()` when task failures need to be observable.
- Always implement the full, bounded shutdown sequence (`shutdown()` → `awaitTermination()` → `shutdownNow()` fallback) during application shutdown.
- For production systems, construct `ThreadPoolExecutor` directly with an explicit, bounded queue and a deliberate rejection policy, rather than relying on `Executors`' convenience defaults.
- Choose the rejection policy deliberately: `CallerRunsPolicy` provides natural backpressure (slowing down the submitter), while `AbortPolicy` (the default) surfaces overload as an immediate, loud exception — pick based on your system's needs.

---

## Performance Notes

- Core vs. maximum pool size, combined with queue capacity, together determine exactly how your system behaves under burst load — this is a genuine, important tuning surface, not a "set it and forget it" decision (fully explored in [[72 - Performance Tuning]]).
- `CallerRunsPolicy` can meaningfully improve system stability under overload by naturally throttling the rate of new task submission (since the submitting thread itself becomes busy running the rejected task), at the cost of temporarily slowing down whatever was submitting tasks.

---

## Comparison Table

|Factory Method|Core=Max?|Queue Type|Risk|
|---|---|---|---|
|`newFixedThreadPool(n)`|Yes (n=n)|Unbounded|Unbounded queue growth under sustained overload|
|`newCachedThreadPool()`|No (0 to Integer.MAX_VALUE)|`SynchronousQueue`|Unbounded thread creation under sustained overload|
|`newSingleThreadExecutor()`|Yes (1=1)|Unbounded|Same queue risk as fixed pool, single-threaded|
|Custom `ThreadPoolExecutor`|Configurable|Configurable (bounded recommended)|Fully controllable|

---

## Summary

- `ExecutorService` is the practical, everyday API for concurrent task execution in Java, extending `Executor` with `submit()`/`Future` support, batch operations, and lifecycle management.
- `execute()` is fire-and-forget with no exception visibility; `submit()` returns a `Future` for results and exception handling.
- `invokeAll()` waits for all tasks; `invokeAny()` returns as soon as any one succeeds.
- The proper shutdown sequence (`shutdown()` → bounded `awaitTermination()` → `shutdownNow()` fallback) is essential — forgetting `shutdown()` entirely is a classic JVM-won't-exit bug.
- For production systems, constructing `ThreadPoolExecutor` directly with a bounded queue and explicit rejection policy is generally recommended over `Executors`' convenience factory methods.

---

## Cheat Sheet

|Method|Purpose|
|---|---|
|`execute(Runnable)`|Fire-and-forget, no result/exception visibility|
|`submit(Callable/Runnable)`|Returns `Future` for result/exception|
|`invokeAll(tasks)`|Waits for ALL tasks to complete|
|`invokeAny(tasks)`|Returns as soon as ANY task succeeds|
|`shutdown()`|Graceful — stop accepting new tasks|
|`shutdownNow()`|Aggressive — interrupt running tasks|
|`awaitTermination(t, unit)`|Wait up to `t` for full shutdown|

---

## Keywords

ExecutorService, ThreadPoolExecutor, Executors, submit, execute, invokeAll, invokeAny, shutdown, RejectedExecutionHandler

---

## Related Notes

[[44 - Thread Pool]] [[45 - Executor Framework]] [[07 - Future]] [[13 - Daemon Thread]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Create a fixed thread pool, submit 5 `Callable` tasks, and print each result via `Future.get()`.
2. Implement the full recommended shutdown sequence and confirm `isTerminated()` returns `true` afterward.

**Medium:** 3. Construct a custom `ThreadPoolExecutor` with a small bounded queue and `AbortPolicy`, then deliberately overload it to trigger `RejectedExecutionException`. 4. Repeat the same overload scenario with `CallerRunsPolicy` instead, and observe the different (non-exception-throwing) behavior.

**Hard:** 5. Build a small benchmark comparing `Executors.newFixedThreadPool()` against a custom bounded `ThreadPoolExecutor` under a sustained burst of tasks exceeding capacity, measuring memory/queue growth in the unbounded case versus controlled rejection/throttling in the bounded case.

---

## Mini Project

**Project: Production-Grade Task Service**

Build a small application-level `TaskService` class that:

1. Internally constructs a `ThreadPoolExecutor` with explicit core/max sizes, a bounded queue, and `CallerRunsPolicy`.
2. Exposes methods for submitting single tasks (returning `Future`) and batches (using `invokeAll()`).
3. Implements a proper `shutdown()` method following the full recommended sequence, callable from an application shutdown hook.
4. Includes a test harness simulating normal load, then a burst exceeding capacity, confirming the rejection policy engages gracefully rather than crashing or growing unbounded.

**Goal:** Build a genuinely production-quality task execution service, directly applying every concept from this note in an integrated, realistic design.

---

## Common Interview Trap

**Trap Question:** _"If I use `Executors.newFixedThreadPool(10)`, and my application suddenly receives a massive burst of 100,000 tasks, will the pool reject any of them, or create more threads to handle the load?"_

The trap is assuming a fixed pool will either scale up or push back under overload. The correct, more precise answer: **neither** — `newFixedThreadPool()` creates a pool where **core size equals maximum size** (both 10), and internally uses an **unbounded** `LinkedBlockingQueue`. Per `ThreadPoolExecutor`'s actual decision logic (core threads first, then queue, then max threads, then reject), once all 10 threads are busy, **every** additional task simply queues up indefinitely — the queue has no capacity limit, so it will **never** reject a task, and it will **never** create more than 10 threads. This means a fixed thread pool provides **no natural backpressure** against a sustained overload — all 100,000 tasks will be accepted and queued, potentially consuming significant memory, with no rejection ever occurring, even if processing them all would take hours. This is precisely why production systems often prefer an explicitly bounded queue (with a deliberate rejection policy) over the unbounded-queue behavior hidden inside `Executors.newFixedThreadPool()`.

---

## Frequently Asked Questions

**Q: Does `shutdown()` wait for the pool to fully terminate before returning?** A: No — `shutdown()` itself returns immediately; use `awaitTermination()` if you need to block until the pool has actually finished.

**Q: What's returned by `shutdownNow()`?** A: A `List<Runnable>` containing the tasks that were still waiting in the queue and never started executing — letting you potentially reschedule or log them.

**Q: Can you call `shutdown()` multiple times safely?** A: Yes — subsequent calls after the first have no additional effect; it's safe, not an error.

**Q: Does `invokeAny()` guarantee the fastest task's result, or just the first to complete without exception?** A: The first to complete **successfully** (without throwing an exception) — if a faster task throws an exception, `invokeAny()` will wait for and return a subsequent successful result instead, only throwing an exception itself if **all** tasks fail.

---

## Revision Checklist

- [ ] I understand the difference between `execute()` and `submit()`, especially regarding exception visibility.
- [ ] I understand the difference between `invokeAll()` and `invokeAny()`.
- [ ] I can implement the complete, recommended shutdown sequence.
- [ ] I understand `ThreadPoolExecutor`'s exact core→queue→max→reject decision logic.
- [ ] I understand why `Executors`' convenience factory methods carry real production risk (unbounded queues/threads) and when to construct `ThreadPoolExecutor` directly instead.

---

## Flashcards

**Q:** What's the key difference between `execute()` and `submit()`? **A:** `submit()` returns a `Future`, letting you retrieve results and observe exceptions; `execute()` is fire-and-forget.

**Q:** What's the difference between `invokeAll()` and `invokeAny()`? **A:** `invokeAll()` waits for every task to complete; `invokeAny()` returns as soon as any one succeeds, cancelling the rest.

**Q:** What is the recommended shutdown sequence? **A:** `shutdown()`, then bounded `awaitTermination()`, falling back to `shutdownNow()` if it times out.

**Q:** What queue type does `Executors.newFixedThreadPool()` use internally? **A:** An unbounded `LinkedBlockingQueue`.

**Q:** What is `CallerRunsPolicy`? **A:** A rejection policy where the submitting thread itself executes the rejected task, providing natural backpressure.

---

## References

- Oracle Java Tutorials — Executors: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.ExecutorService` API Documentation
- `java.util.concurrent.ThreadPoolExecutor` API Documentation
- `java.util.concurrent.Executors` API Documentation