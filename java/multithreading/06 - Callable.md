# Callable

## Objective

By the end of this note, you will understand:

- What the `Callable` interface is and why it was introduced in Java 5
- How `Callable` differs from `Runnable` in practice, not just in theory
- How `Callable` is used together with `ExecutorService` and `Future`
- How to properly retrieve results and handle exceptions from concurrent tasks

This note builds directly on [[05 - Runnable]] and sets up [[07 - Future]] and [[08 - FutureTask]].

---

## Why should I learn this?

`Runnable` has a major limitation: it cannot return a value, and it cannot throw checked exceptions. Real-world concurrent tasks very often **need** to return something — the result of a database query, the result of an API call, a computed value. `Callable` was introduced specifically to solve this gap.

If you don't understand `Callable`, you'll be tempted to work around `Runnable`'s limitations using shared mutable fields (an instance variable to "smuggle" a result out) — which introduces exactly the kind of race conditions and thread-safety bugs this entire vault is trying to teach you to avoid.

`Callable` is also a core building block for `Future` (see [[07 - Future]]) and `CompletableFuture` (see [[49 - CompletableFuture]]), both heavily used in real production asynchronous code.

---

## Beginner Explanation

Think back to the `Runnable` analogy: a sticky note describing a task, with no way to report back a result.

`Callable` is like a sticky note that says: **"Do this task, and then write your answer on the back of this note before you're done."**

In code terms:

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

- `call()` (not `run()`) is the method you implement.
- It **returns a value** of type `V` (a generic type — you decide what it is: `Integer`, `String`, a custom object, etc.).
- It **can throw a checked exception** (`Exception`), unlike `Runnable.run()`.

So whenever your task needs to compute or fetch something and hand that result back to whoever asked for it, `Callable` is the right tool.

---

## Real World Analogy

**Courier Service Analogy (extended from [[05 - Runnable]])**

Recall the courier analogy: a `Runnable` is like handing a courier a package to deliver, with no expectation of anything coming back.

A `Callable` is like sending a courier on an **errand that produces a result** — for example, sending them to the bank to withdraw cash and bring it back to you. The courier:

1. Goes and performs the task (just like `Runnable`).
2. **Comes back with something** — the withdrawn cash (the return value).
3. If something goes wrong during the errand (the bank is closed, an ID is rejected), they can **report back the problem** (a checked exception), rather than silently failing.

This "go do something, and bring back either a result or a reported problem" pattern is exactly what `Callable` represents, and it maps directly onto how `Future` (see [[07 - Future]]) is used to retrieve that eventual result.

---

## Technical Explanation

### The Interface Definition

```java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

- **Generic type `V`** — the type of the result. E.g., `Callable<Integer>`, `Callable<String>`, `Callable<List<Order>>`.
- **`call()`, not `run()`** — a different method name from `Runnable`, deliberately, so the two interfaces don't collide.
- **`throws Exception`** — `call()` is allowed to throw **any** checked exception, unlike `Runnable.run()`.

### Why `Callable` exists (historical context)

`Callable` was introduced in **Java 5** (2004), alongside the entire `java.util.concurrent` package (`ExecutorService`, `Future`, thread pools, etc.). Before Java 5, developers had to manage raw `Thread` objects manually and had no clean, standard way to get a return value out of a background task. `Callable` (together with `Future`) filled this critical gap.

### How `Callable` is used with `ExecutorService`

`Callable` cannot be executed directly by a raw `Thread` (the `Thread` constructor only accepts `Runnable`). Instead, `Callable` is designed to work with `ExecutorService`:

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Callable<Integer> task = () -> {
    Thread.sleep(1000); // simulate work
    return 42;
};

Future<Integer> future = executor.submit(task);   // submit returns a Future
Integer result = future.get();                    // blocks until result is ready

executor.shutdown();
```

We'll go deeper into `Future` in the next note ([[07 - Future]]) — for now, notice that:

- `executor.submit(Callable<V>)` returns a `Future<V>`.
- `future.get()` **blocks the calling thread** until the result is available, then returns it (or throws an exception if the task failed).

### Checked Exception Handling

Because `call()` can throw `Exception`, any checked exception thrown inside the task is **not lost** — it gets wrapped inside an `ExecutionException` and re-thrown when you call `future.get()`:

```java
try {
    Integer result = future.get();
} catch (ExecutionException e) {
    Throwable actualCause = e.getCause(); // the original exception thrown inside call()
    System.out.println("Task failed: " + actualCause.getMessage());
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

This is a major advantage over `Runnable`, where an exception thrown inside `run()` would just be handled by the thread's default uncaught exception handler (usually just printed to `System.err`), with no clean way for the calling code to catch and react to it.

---

## Internal Working

Step-by-step of what happens when you submit a `Callable` to an `ExecutorService`:

1. You create a `Callable<V>` instance, implementing your logic inside `call()`.
2. You call `executor.submit(callableTask)`.
3. The `ExecutorService` wraps your `Callable` inside an internal task-tracking object (commonly a `FutureTask`, see [[08 - FutureTask]]) and returns a `Future<V>` object to you immediately — **without blocking**.
4. Behind the scenes, one of the executor's worker threads eventually picks up this wrapped task and calls its `call()` method.
5. If `call()` completes normally, the returned value is stored inside the `Future` object, and its state becomes "completed."
6. If `call()` throws an exception, that exception is captured and stored inside the `Future` object instead, and its state becomes "completed exceptionally."
7. When your code later calls `future.get()`, it either:
    - Returns the stored result immediately (if the task is already done), or
    - **Blocks** the calling thread until the task finishes, then returns the result or throws `ExecutionException` wrapping the original failure.

---

## Memory Diagram

```
     Calling Thread                    Executor's Worker Thread
   -------------------                -----------------------------
   | Callable<V> task |  submit()  ->  | Executes task.call()      |
   -------------------                |   - success -> store result|
           |                          |   - failure -> store cause |
           v                          -----------------------------
   Future<V> object returned                       |
   (immediately, non-blocking)                     |
           |                                        |
           |  future.get() called later             |
           |<---------------------------------------|
           v
   Result returned OR ExecutionException thrown
```

---

## Execution Flow

```
Define Callable<V> task
        |
        v
executor.submit(task)
        |
        v
Future<V> returned immediately (task not necessarily done yet)
        |
        v
Worker thread (eventually) executes task.call()
        |
   -----------------------
   |                     |
   v                     v
Success:            Failure:
store result        store exception
in Future           in Future
        |                     |
        --------- future.get() ---------
                       |
                       v
        Returns result, OR throws
        ExecutionException wrapping
        the original exception
```

---

## Code Example

```java
import java.util.concurrent.*;

public class CallableDemo {

    public static void main(String[] args) {

        ExecutorService executor = Executors.newFixedThreadPool(2);

        // Callable that returns a computed value
        Callable<Integer> sumTask = () -> {
            int sum = 0;
            for (int i = 1; i <= 100; i++) {
                sum += i;
            }
            return sum;
        };

        // Callable that throws a checked exception
        Callable<String> riskyTask = () -> {
            if (true) {
                throw new java.io.IOException("Simulated I/O failure");
            }
            return "This will never be returned";
        };

        try {
            Future<Integer> sumFuture = executor.submit(sumTask);
            Future<String> riskyFuture = executor.submit(riskyTask);

            System.out.println("Sum result: " + sumFuture.get()); // blocks until ready

            try {
                riskyFuture.get(); // this will throw
            } catch (ExecutionException e) {
                System.out.println("Caught expected failure: " + e.getCause().getMessage());
            }

        } catch (InterruptedException | ExecutionException e) {
            Thread.currentThread().interrupt();
        } finally {
            executor.shutdown();
        }
    }
}
```

### Line-by-line explanation

1. `Callable<Integer> sumTask = () -> {...}` — a task that computes and **returns** a value (`1+2+...+100 = 5050`).
2. `Callable<String> riskyTask = () -> {...}` — a task that **throws a checked exception** (`IOException`) — something `Runnable` could never do directly.
3. `executor.submit(sumTask)` — submits the task and gets back a `Future<Integer>` **immediately** (non-blocking); the task may not have even started yet.
4. `sumFuture.get()` — this line **blocks** the main thread until the sum task finishes, then returns `5050`.
5. `riskyFuture.get()` — this line throws an `ExecutionException`, because the underlying task threw an `IOException`. We catch it and inspect `.getCause()` to get the **original** exception that was thrown inside `call()`.
6. `executor.shutdown()` — cleanly shuts down the thread pool once we're done (see [[46 - ExecutorService]] for full lifecycle details).

---

## Dry Run

|Step|Action|State|
|---|---|---|
|1|`submit(sumTask)` called|Future A returned, task running/queued|
|2|`submit(riskyTask)` called|Future B returned, task running/queued|
|3|Worker thread executes `sumTask.call()`|Returns 5050, stored in Future A|
|4|`sumFuture.get()` called|Blocks briefly if not done, then returns 5050|
|5|Worker thread executes `riskyTask.call()`|Throws `IOException`, stored in Future B|
|6|`riskyFuture.get()` called|Throws `ExecutionException`, `.getCause()` = the `IOException`|

---

## Common Mistakes

1. **Assuming `submit(Callable)` blocks immediately.**
    
    - It does not — `submit()` returns a `Future` right away. Blocking only happens when you call `.get()` on that `Future`.
2. **Forgetting that exceptions from `call()` are wrapped in `ExecutionException`.**
    
    - Beginners often try to catch the original exception type directly (e.g., `catch (IOException e)`) around `future.get()`, but the actual exception thrown by `.get()` is `ExecutionException`; you must call `.getCause()` to get the original exception.
3. **Using `Runnable` with a mutable field to fake a return value, instead of using `Callable`.**
    
    - This pattern (`Runnable` writing its result into a shared instance variable) is unnecessarily fragile and thread-unsafe compared to simply using `Callable` and `Future`, which handle this safely and idiomatically.
4. **Forgetting to call `executor.shutdown()`.**
    
    - Just like with any `ExecutorService`, forgetting to shut it down can leave the JVM process alive indefinitely, since the pool's threads are typically non-daemon threads (see [[46 - ExecutorService]] for the full lifecycle).

---

## Production Usage

- **Parallel data fetching**: E.g., calling 3 different microservices concurrently, each returning a piece of data needed to assemble a final response — each call modeled as a `Callable<Response>`.
- **Batch processing with results**: Processing a batch of records where each unit of work returns a status or result object, later aggregated after all `Future`s complete.
- **Timeout-bound operations**: `Future.get(timeout, unit)` (an overloaded version) is commonly used in production to bound how long the calling code will wait for a slow external call before giving up.
- **Spring's `@Async` methods with return types**: When an `@Async` method declares a return type like `CompletableFuture<T>`, Spring internally represents the underlying work similarly to a `Callable`, since it needs both a return value and exception propagation.

---

## Interview Questions

**Beginner:**

1. What is the difference between `Runnable` and `Callable`?
2. What method does `Callable` define, and what does it return?
3. Can `Callable`'s method throw checked exceptions?

**Intermediate:** 4. How do you submit a `Callable` to an `ExecutorService`, and what do you get back? 5. What happens if the task inside a `Callable` throws an exception — how is that exception surfaced to the caller? 6. Why can't you pass a `Callable` directly to a `Thread` constructor?

**Advanced:** 7. What is the relationship between `Callable`, `Future`, and `FutureTask`? (Full detail in [[07 - Future]] and [[08 - FutureTask]].) 8. Why was `Callable` introduced in Java 5 instead of just extending `Runnable` to support return values?

**Production-Level:** 9. In a system that calls 3 external services in parallel and combines their results, how would you use `Callable` and `Future` (or `CompletableFuture`) to implement this cleanly? 10. How would you implement a timeout so that if one of several parallel `Callable` tasks takes too long, the rest of the system doesn't wait indefinitely?

---

## Follow-up Questions

- If you call `future.get()` on a `Callable`-based task that never finishes, what happens? _(The calling thread blocks indefinitely, unless you use the overloaded `get(long timeout, TimeUnit unit)` version, which throws a `TimeoutException` after the specified wait time.)_
- Can `Callable` be used with a plain `Thread` object at all? _(Not directly — you'd need to wrap it in a `FutureTask`, which implements `Runnable`, allowing it to be passed to a `Thread`. See [[08 - FutureTask]].)_
- Is `Callable` a functional interface, like `Runnable`? _(Yes — it has one abstract method, `call()`, so it supports lambda expressions just like `Runnable` does.)_

---

## Best Practices

- Use `Callable` whenever your task needs to **return a result** or **may throw a checked exception** — don't force-fit `Runnable` with mutable shared fields as a workaround.
- Always handle `ExecutionException` properly by inspecting `.getCause()` to get the original failure reason.
- Use the timeout-based overload of `Future.get(timeout, unit)` in production code where waiting indefinitely for a task is unacceptable.
- Prefer `CompletableFuture` (see [[49 - CompletableFuture]]) over raw `Callable`/`Future` when you need to chain multiple asynchronous operations together — raw `Future` does not support composition well.

---

## Performance Notes

- `Callable` itself has no meaningful performance overhead compared to `Runnable` — the difference is purely about API capability (return values, exceptions), not runtime cost.
- Blocking on `future.get()` **ties up the calling thread** while it waits — in high-throughput systems, excessive blocking on `Future.get()` calls can itself become a bottleneck; this is one of the motivations behind non-blocking approaches like `CompletableFuture` callbacks.

---

## Comparison Table

|Aspect|`Runnable`|`Callable<V>`|
|---|---|---|
|Method|`void run()`|`V call() throws Exception`|
|Return value|No|Yes (generic type `V`)|
|Checked exceptions|Not allowed|Allowed|
|Can be passed to `Thread` constructor?|Yes|No (needs `FutureTask` wrapper)|
|Can be submitted to `ExecutorService`?|Yes (`execute`/`submit`)|Yes (`submit`, returns `Future<V>`)|
|Introduced in|Java 1.0|Java 5|

---

## Summary

- `Callable<V>` is a functional interface with a single method, `V call() throws Exception`.
- Unlike `Runnable`, it can return a result and throw checked exceptions.
- `Callable` is designed to be used with `ExecutorService.submit()`, which returns a `Future<V>` representing the eventual result.
- Exceptions thrown inside `call()` are captured and re-thrown (wrapped in `ExecutionException`) when `future.get()` is called.
- `Callable` cannot be passed directly to a `Thread` constructor — only `ExecutorService` accepts it directly.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|`Callable<V>`|Functional interface: `V call() throws Exception`|
|Return value?|Yes|
|Checked exceptions?|Yes|
|Submit to executor|`Future<V> f = executor.submit(callable);`|
|Get result|`V result = f.get();` (blocking)|
|Exception on failure|`ExecutionException`, use `.getCause()`|

---

## Keywords

Callable, Future, ExecutionException, ExecutorService, Checked Exception, Generic Type, Blocking Call, Java 5

---

## Related Notes

[[05 - Runnable]] [[07 - Future]] [[08 - FutureTask]] [[46 - ExecutorService]] [[49 - CompletableFuture]]

---

## Practice Exercises

**Easy:**

1. Write a `Callable<Integer>` that returns the square of a number, submit it to an `ExecutorService`, and print the result.
2. Write a `Callable<String>` that throws a checked exception, and catch the resulting `ExecutionException`, printing the original cause.

**Medium:** 3. Submit 5 different `Callable<Integer>` tasks (each computing something different) to a fixed thread pool of size 2, collect all their `Future`s in a list, then loop through and print each result using `.get()`. 4. Modify the exercise above to use `future.get(500, TimeUnit.MILLISECONDS)` for one deliberately slow task, and handle the resulting `TimeoutException`.

**Hard:** 5. Build a small program that calls 3 "simulated external services" (each a `Callable` with an artificial delay) in parallel, waits for all of them using their `Future`s, and combines their results into a single summary object — a simplified preview of real-world parallel API aggregation.

---

## Mini Project

**Project: Parallel Price Checker**

Build a small console application that:

1. Defines 3 `Callable<Double>` tasks, each simulating a "price check" from a different vendor (using `Thread.sleep` to simulate network delay, and returning a random price).
2. Submits all 3 to an `ExecutorService` with a fixed thread pool.
3. Waits for all 3 results using their `Future` objects.
4. Prints the lowest price found among the three vendors, along with the total time taken (which should be close to the slowest single task's time, not the sum of all three — demonstrating the concurrency benefit from [[03 - Why Multithreading Exists]]).

**Goal:** Apply `Callable` and `Future` together in a realistic, small-scale parallel-computation scenario.

---

## Common Interview Trap

**Trap Question:** _"If I call `future.get()` right after `executor.submit(callable)`, is there any real concurrency happening?"_

The trap here is that many candidates assume calling `.get()` immediately negates any benefit of `Callable`/`ExecutorService`, since it blocks right away. The nuanced answer: if you submit **multiple** tasks first, and only call `.get()` on each **after** all submissions are done, all tasks run concurrently in the background while your calling thread is free (or busy submitting more tasks) — only the `.get()` calls themselves block, and only for tasks not yet finished. Submitting one task and immediately blocking on its result provides no concurrency benefit; the value of `Callable`/`Future` comes from submitting multiple tasks upfront and collecting results later.

---

## Frequently Asked Questions

**Q: Can I use a lambda expression for `Callable`, just like with `Runnable`?** A: Yes — `Callable` is also a functional interface, so lambdas work directly: `Callable<Integer> c = () -> 42;`

**Q: What's the difference between `ExecutionException` and the original exception thrown inside `call()`?** A: `ExecutionException` is a **wrapper** — it's thrown by `future.get()` itself, and its `.getCause()` method returns the actual exception that was thrown inside your `call()` method.

**Q: Does `Callable` exist for use only with `ExecutorService`?** A: Primarily, yes — this is its main intended use case, though it can also be manually wrapped in a `FutureTask` (which implements `Runnable`) to be used with a raw `Thread` (see [[08 - FutureTask]]).

**Q: Is `Callable` better than `Runnable` in general?** A: Not universally "better" — they serve different purposes. Use `Runnable` for fire-and-forget tasks with no result; use `Callable` when you need a return value or need to propagate a checked exception.

---

## Revision Checklist

- [ ] I can explain the difference between `Runnable` and `Callable`.
- [ ] I know that `Callable` can return a value and throw checked exceptions.
- [ ] I understand that `Callable` cannot be passed directly to a `Thread` constructor.
- [ ] I know how to submit a `Callable` to an `ExecutorService` and retrieve its result via `Future`.
- [ ] I understand how exceptions from `call()` are wrapped in `ExecutionException`.

---

## Flashcards

**Q:** What method does `Callable<V>` define? **A:** `V call() throws Exception`

**Q:** Can `Callable` throw checked exceptions? **A:** Yes — unlike `Runnable`.

**Q:** What does `ExecutorService.submit(Callable<V>)` return? **A:** A `Future<V>` representing the eventual result.

**Q:** How do you get the original exception thrown inside `call()` after catching an `ExecutionException`? **A:** Call `.getCause()` on the caught `ExecutionException`.

**Q:** In which Java version was `Callable` introduced? **A:** Java 5, alongside the `java.util.concurrent` package.

---

## References

- Oracle Java Tutorials — Executors: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.Callable` API Documentation
- `java.util.concurrent.Future` API Documentation
- JSR 166 (Concurrency Utilities), the JSR that introduced `java.util.concurrent` in Java 5