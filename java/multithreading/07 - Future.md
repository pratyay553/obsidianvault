# Future

## Objective

By the end of this note, you will understand:

- What a `Future` object represents and why it's needed
- How `Future` relates to `Callable` and `ExecutorService`
- How to check task status, cancel tasks, and retrieve results with/without timeouts
- The key limitations of `Future` that later led to `CompletableFuture`

This note builds directly on [[06 - Callable]] and prepares you for [[08 - FutureTask]] and [[49 - CompletableFuture]].

---

## Why should I learn this?

When you submit a `Callable` task to an `ExecutorService`, the task might take time to complete — but your code needs a way to say **"give me a handle to that eventual result, and let me check on it or wait for it later."** That handle is exactly what `Future` provides.

Understanding `Future` is essential because:

- It's the standard mechanism for retrieving results from asynchronous tasks in the classic `java.util.concurrent` API
- It's frequently tested in interviews, especially around blocking behavior and cancellation
- Understanding its limitations is what makes `CompletableFuture` (see [[49 - CompletableFuture]]) click intuitively later — you'll appreciate exactly what problems it was designed to solve

---

## Beginner Explanation

Imagine you drop off your car at a repair shop in the morning. The shop gives you a **claim ticket** — a small piece of paper with a number on it. That ticket:

- Doesn't contain your repaired car
- Doesn't tell you exactly when the car will be ready
- Lets you come back **later** and either check if it's ready, or wait around until it is

A `Future` is exactly like that claim ticket. When you submit a task to an `ExecutorService`, you don't get the result immediately — you get a `Future`, a "claim ticket" for a result that will exist **at some point in the future**. You can then:

- Check if it's done yet (`isDone()`)
- Wait for it to finish and grab the result (`get()`)
- Cancel it if you no longer need it (`cancel()`)

---

## Real World Analogy

**Airport Analogy**

Think of checking in your luggage at an airport:

- You hand over your suitcase (the task) at check-in.
- You get a **baggage claim ticket** (the `Future`) in return.
- You don't get your suitcase back immediately — you go through security, board your flight, etc. (your program continues doing other things).
- At your destination, you go to baggage claim and **wait** for your specific suitcase to appear (`future.get()` — blocking until it's ready).
- If your suitcase is lost or damaged, you're informed of the problem (an exception, wrapped in `ExecutionException`) instead of getting your bag.
- You could also decide, mid-flight, that you don't want to wait for that bag anymore and abandon the claim (`future.cancel()`), although this doesn't always succeed once baggage handling has already started (just like cancelling a task that has already started running isn't always guaranteed to stop it).

---

## Technical Explanation

### The Interface Definition

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}
```

Let's break each method down:

|Method|Purpose|
|---|---|
|`cancel(boolean mayInterruptIfRunning)`|Attempts to cancel the task. If `true`, and the task is already running, the executing thread is interrupted (see [[21 - Interrupt()]]). Returns `true` if cancellation succeeded.|
|`isCancelled()`|Returns `true` if the task was cancelled before completing normally.|
|`isDone()`|Returns `true` if the task completed — whether normally, exceptionally, or via cancellation.|
|`get()`|**Blocks** the calling thread until the result is available, then returns it (or throws an exception).|
|`get(long timeout, TimeUnit unit)`|Same as `get()`, but throws `TimeoutException` if the result isn't ready within the given time.|

### How a `Future` is obtained

`Future` objects are returned by `ExecutorService` submission methods:

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> future = executor.submit(() -> {
    Thread.sleep(2000);
    return 100;
});
```

At this point, `future` exists **immediately**, even though the task itself may not have started or finished yet.

### Blocking vs Non-blocking checks

```java
if (future.isDone()) {
    System.out.println("Already done: " + future.get());
} else {
    System.out.println("Still working...");
}
```

`isDone()` is a **non-blocking** check — it just tells you the current status without waiting. In contrast, `get()` is **blocking** — it will pause the calling thread until the result is ready (or forever, if you don't use the timeout overload and the task never completes).

### Cancelling a Task

```java
boolean cancelled = future.cancel(true); // true = interrupt if already running
```

- If the task **hasn't started yet**, cancellation typically succeeds cleanly — it's simply removed from the queue.
- If the task **is already running**, cancellation only works if `mayInterruptIfRunning` is `true`, AND the task's code actually **respects thread interruption** (see [[21 - Interrupt()]]) — otherwise, the task keeps running to completion regardless.
- If the task **has already completed**, `cancel()` returns `false` — you can't cancel something that's already done.

---

## Internal Working

Step-by-step of the full `Future` lifecycle:

1. `executor.submit(callableOrRunnable)` is called.
2. Internally, the executor wraps the task in a `FutureTask` (see [[08 - FutureTask]]), which implements both `Runnable` (so a worker thread can run it) and `Future` (so the calling code can query/retrieve it).
3. The `FutureTask` is placed into the executor's internal work queue.
4. A worker thread eventually picks it up and executes it.
5. While the task is running, calls to `future.isDone()` return `false`, and calls to `future.get()` **block**.
6. When the task finishes:
    - If successful, the result is stored internally, `isDone()` becomes `true`, and any blocked `get()` calls unblock and return the result.
    - If it threw an exception, that exception is stored internally instead, and `get()` will throw `ExecutionException` wrapping it.
    - If it was cancelled before starting or during execution (with interruption), `isCancelled()` becomes `true`, and `get()` throws `CancellationException`.

---

## Memory Diagram

```
   Calling Thread                     Future<V> object (shared state)
 -------------------               ---------------------------------
 | submit(task)     | ----------> | state: PENDING                 |
 -------------------               ---------------------------------
         |                                     ^
         | future.get()  (BLOCKS)              |  worker thread eventually
         |                                     |  updates this state
         v                                     |
 -------------------               ---------------------------------
 | Waiting...       |    <------- | state: COMPLETED (result or     |
 -------------------               |         exception stored)      |
         |                          ---------------------------------
         v
  Result returned OR exception thrown
```

---

## Execution Flow

```
submit(Callable/Runnable)
        |
        v
Future<V> returned immediately
        |
        |------------------------------|
        |  Caller can do other work    |   Worker thread executes
        |  OR call future.get() now    |   the task in background
        |------------------------------|
        |
        v
future.get() called (blocks if not done)
        |
   -----------------------------
   |            |              |
   v            v              v
Success    Exception       Cancelled
(returns   (throws         (throws
 result)   ExecutionExc.)  CancellationExc.)
```

---

## Code Example

```java
import java.util.concurrent.*;

public class FutureDemo {

    public static void main(String[] args) throws Exception {

        ExecutorService executor = Executors.newFixedThreadPool(2);

        Future<Integer> future = executor.submit(() -> {
            System.out.println("Task started on: " + Thread.currentThread().getName());
            Thread.sleep(2000); // simulate slow work
            return 123;
        });

        System.out.println("Task submitted. isDone right away? " + future.isDone());

        // Do other useful work while the task runs in the background
        System.out.println("Main thread doing other work while waiting...");

        // Now block and wait for the result, with a timeout
        try {
            Integer result = future.get(3, TimeUnit.SECONDS);
            System.out.println("Result: " + result);
        } catch (TimeoutException e) {
            System.out.println("Task took too long!");
        } catch (ExecutionException e) {
            System.out.println("Task failed: " + e.getCause());
        }

        System.out.println("isDone after get(): " + future.isDone());

        executor.shutdown();
    }
}
```

### Line-by-line explanation

1. `executor.submit(...)` — submits the task and immediately returns a `Future<Integer>`, without waiting for the task to finish.
2. `future.isDone()` right after submission — almost certainly prints `false`, since the task sleeps for 2 seconds and has likely not even started yet.
3. The main thread prints "doing other work" — demonstrating that submitting a task does **not** block the caller.
4. `future.get(3, TimeUnit.SECONDS)` — blocks for **up to 3 seconds**, waiting for the 2-second task to finish. Since 2 < 3, this should succeed and return `123`.
5. If the task had taken longer than 3 seconds, a `TimeoutException` would be thrown instead — the calling thread would **not** wait forever.
6. `future.isDone()` after `get()` — now prints `true`, since the task has completed by this point.

---

## Dry Run

|Time (approx)|Event|
|---|---|
|0.0s|Task submitted, `Future` returned|
|0.0s|`isDone()` checked → `false`|
|0.0s|Main thread prints "doing other work"|
|0.0s → 2.0s|Worker thread executes task, sleeping|
|2.0s|Task completes, result (123) stored in `Future`|
|2.0s|`future.get(3, SECONDS)` (called at ~0.0s) unblocks, returns 123|
|2.0s|`isDone()` checked again → `true`|

---

## Common Mistakes

1. **Calling `future.get()` immediately after `submit()`, without doing anything else in between.**
    
    - This defeats the purpose of asynchronous execution — if you submit a task and then instantly block on its result with nothing else happening concurrently, you've gained no concurrency benefit over just calling the method directly. `Future` is most useful when you submit **multiple** tasks first, then collect results later.
2. **Using `get()` without a timeout in production code.**
    
    - If the underlying task hangs indefinitely (e.g., stuck waiting on a dead external service), `future.get()` will block **forever**, potentially freezing the calling thread and cascading into larger system issues. Production code should almost always use the timeout overload.
3. **Assuming `cancel(true)` always stops a running task.**
    
    - Cancellation only works reliably for tasks that **check** their interrupted status or handle `InterruptedException` properly (see [[21 - Interrupt()]]). A task in a tight, non-interruptible loop will ignore cancellation and run to completion regardless.
4. **Forgetting that `Future` provides no way to chain or combine multiple asynchronous results cleanly.**
    
    - If you need to run task A, then use its result to start task B, or combine results from multiple `Future`s, raw `Future` makes this awkward and blocking-heavy. This exact limitation is what `CompletableFuture` (see [[49 - CompletableFuture]]) was designed to solve.

---

## Production Usage

- **Parallel API calls with timeouts**: A backend service calling 3 downstream services in parallel, using `Future.get(timeout, unit)` on each to avoid waiting forever on a slow or unresponsive dependency.
- **Batch job result aggregation**: Submitting many `Callable` chunks of a large dataset for parallel processing, then collecting all `Future` results once processing completes.
- **Cancellable background tasks**: User-triggered operations (e.g., "cancel my search request") that can be cancelled via `Future.cancel()` if the user navigates away before the operation completes.
- **Legacy codebases**: Many enterprise Java systems written before `CompletableFuture` became common (pre-Java 8, or early Java 8 codebases) still rely heavily on raw `Future`, so understanding it is important for working with existing production code.

---

## Interview Questions

**Beginner:**

1. What is a `Future`, and what does it represent?
2. Does calling `executor.submit()` block the calling thread?
3. What does `future.get()` do if the task isn't finished yet?

**Intermediate:** 4. What is the difference between `future.get()` and `future.get(timeout, unit)`? 5. What exception is thrown if you call `.get()` on a `Future` whose task threw an exception? 6. What does `isDone()` return for a cancelled task?

**Advanced:** 7. Under what conditions does `future.cancel(true)` actually succeed in stopping an already-running task? 8. Why is raw `Future` considered limited for chaining multiple asynchronous operations together?

**Production-Level:** 9. Why is it considered risky to call `future.get()` without a timeout in a production service handling user requests? 10. How would you design a system that calls 5 external services in parallel, with an overall maximum wait time of 2 seconds, using `Future`?

---

## Follow-up Questions

- If you never call `.get()` on a `Future`, does the underlying task still execute? _(Yes — the task runs regardless of whether you ever retrieve its result; `Future` is just a handle for retrieval, not a trigger for execution.)_
- Can multiple threads call `.get()` on the same `Future` object? _(Yes — this is safe; all callers will receive the same result once it's available, or all will block until it is.)_
- What happens if you call `.cancel()` on a `Future` whose task has already completed? _(It returns `false` — cancellation only affects tasks that haven't completed yet.)_

---

## Best Practices

- Always prefer the timeout overload, `get(timeout, unit)`, in production code, to avoid indefinite blocking.
- Submit multiple tasks first, then collect their `Future` results afterward, to actually benefit from concurrency.
- Ensure long-running task logic periodically checks `Thread.currentThread().isInterrupted()` so that `cancel(true)` can actually take effect (see [[21 - Interrupt()]]).
- For anything beyond simple "submit and later retrieve" use cases (chaining, combining, non-blocking callbacks), prefer `CompletableFuture` over raw `Future` (see [[49 - CompletableFuture]]).

---

## Performance Notes

- `Future.get()` **blocks the calling thread**, which means that thread cannot do other useful work while waiting — in high-throughput systems, excessive blocking on `Future.get()` can itself limit scalability.
- Cancellation, when honored properly, can save significant CPU/resources by stopping unnecessary work early — but only if task code is written to respect interruption.
- Holding onto many `Future` objects simultaneously (e.g., in a large batch job) is fine memory-wise, but blocking on all of them sequentially (instead of using something like `invokeAll` or asynchronous composition) can needlessly serialize what should be concurrent waiting.

---

## Comparison Table

|Aspect|`Future`|`CompletableFuture` (see [[49 - CompletableFuture]])|
|---|---|---|
|Introduced in|Java 5|Java 8|
|Blocking retrieval|Yes (`get()`)|Optional — also supports non-blocking callbacks|
|Chaining multiple async operations|Awkward / not supported|Built-in (`thenApply`, `thenCompose`, etc.)|
|Combining multiple futures|Manual, verbose|Built-in (`allOf`, `anyOf`)|
|Manually completing it|Not supported|Supported (`complete()`)|

---

## Summary

- A `Future<V>` represents a "claim ticket" for a result that will be available at some point in the future.
- It's returned immediately by `ExecutorService.submit()`, without blocking the calling thread.
- `get()` blocks until the result is ready (or throws an exception); the timeout overload avoids waiting forever.
- Cancellation via `cancel()` only reliably works for tasks that haven't started, or tasks that properly respect thread interruption.
- Raw `Future` has real limitations around chaining and combining asynchronous operations — this gap is what `CompletableFuture` was built to close.

---

## Cheat Sheet

|Method|Behavior|
|---|---|
|`isDone()`|Non-blocking check if task finished (success, failure, or cancelled)|
|`isCancelled()`|Non-blocking check if task was cancelled|
|`get()`|Blocks until result is ready; throws `ExecutionException` on task failure|
|`get(timeout, unit)`|Same as `get()`, but throws `TimeoutException` if not ready in time|
|`cancel(true)`|Attempts to cancel; interrupts if already running|
|`cancel(false)`|Attempts to cancel only if not yet started|

---

## Keywords

Future, ExecutorService, ExecutionException, TimeoutException, CancellationException, Blocking, Non-blocking, Cancellation

---

## Related Notes

[[06 - Callable]] [[08 - FutureTask]] [[46 - ExecutorService]] [[49 - CompletableFuture]] [[21 - Interrupt()]]

---

## Practice Exercises

**Easy:**

1. Submit a `Callable` task to an `ExecutorService`, immediately check `isDone()`, then call `get()` and print the result.
2. Modify the exercise to use `get(1, TimeUnit.SECONDS)` on a task that takes 3 seconds, and handle the resulting `TimeoutException`.

**Medium:** 3. Submit 5 `Callable` tasks with varying artificial delays, collect all their `Future` objects in a list, and print all results only after all submissions are done (to properly benefit from concurrency). 4. Try cancelling a task before it starts (immediately after submission) and print whether cancellation succeeded.

**Hard:** 5. Write a long-running task that periodically checks `Thread.currentThread().isInterrupted()` inside a loop, submit it, then call `future.cancel(true)` from the main thread after 1 second. Confirm the task actually stops early, rather than running to completion.

---

## Mini Project

**Project: Multi-Service Aggregator with Timeout**

Build a small console application that:

1. Simulates 4 "external service calls," each a `Callable<String>` with a random delay between 500ms and 3000ms.
2. Submits all 4 to an `ExecutorService`.
3. Waits for each result using `future.get(1500, TimeUnit.MILLISECONDS)`.
4. For services that respond in time, prints their result; for services that time out, prints a fallback message instead (e.g., "Service X unavailable, using default").
5. Prints the total time taken for the whole operation.

**Goal:** Simulate a realistic production scenario where you can't let one slow dependency block your entire response indefinitely.

---

## Common Interview Trap

**Trap Question:** _"If I call `future.cancel(true)` on a task, is it guaranteed to stop immediately?"_

The trap is assuming `cancel()` behaves like a hard "kill switch." In reality, cancellation is **cooperative**: if the task hasn't started, it's simply removed from the queue and won't run at all. But if it's **already running**, `cancel(true)` merely sends an **interrupt signal** to the executing thread — the task's own code must be written to detect this (via `InterruptedException` or checking `isInterrupted()`) and voluntarily stop. A task stuck in a tight computational loop that never checks its interrupted status will simply **keep running to completion**, ignoring the cancellation request entirely.

---

## Frequently Asked Questions

**Q: Does submitting a task to `ExecutorService` guarantee it starts running immediately?** A: No — if all worker threads in the pool are busy, the task waits in an internal queue until a thread becomes available.

**Q: What happens if I call `.get()` twice on the same `Future`?** A: Both calls return the same result (or throw the same exception) — calling `.get()` multiple times is safe and doesn't re-run the task.

**Q: Is `Future` thread-safe to share across multiple threads?** A: Yes — multiple threads can safely call methods like `.get()`, `.isDone()`, or `.cancel()` on the same `Future` instance concurrently.

**Q: Why would I ever use raw `Future` instead of `CompletableFuture` in new code?** A: In most modern code, `CompletableFuture` is preferred for its flexibility. However, raw `Future` is simpler for basic "submit and later retrieve" use cases, and you'll still encounter it heavily in legacy codebases and certain library APIs that predate `CompletableFuture`.

---

## Revision Checklist

- [ ] I understand what a `Future` represents and how it relates to `Callable`/`ExecutorService`.
- [ ] I know the difference between `get()` and `get(timeout, unit)`.
- [ ] I understand how exceptions from the task propagate through `Future.get()`.
- [ ] I understand that cancellation is cooperative, not forced.
- [ ] I understand the key limitations of `Future` that motivated `CompletableFuture`.

---

## Flashcards

**Q:** What does a `Future<V>` represent? **A:** A handle to a result that will become available at some point in the future.

**Q:** Does `future.get()` block the calling thread? **A:** Yes, until the result is ready (unless using the timeout overload, which throws `TimeoutException` instead of waiting forever).

**Q:** What exception wraps a failure that occurred inside the original task? **A:** `ExecutionException` — use `.getCause()` to get the original exception.

**Q:** Is cancellation of a running task guaranteed to succeed? **A:** No — it only works if the task's code respects thread interruption.

**Q:** What later API addressed `Future`'s limitations around chaining and combining async operations? **A:** `CompletableFuture`, introduced in Java 8.

---

## References

- Oracle Java Tutorials — Executors: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.Future` API Documentation
- `java.util.concurrent.ExecutorService` API Documentation