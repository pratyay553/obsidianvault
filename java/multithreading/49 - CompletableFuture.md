# CompletableFuture

## Objective

By the end of this note, you will understand:

- Exactly what limitations of `Future` (from [[07 - Future]]) `CompletableFuture` solves
- How to chain, combine, and compose asynchronous operations without blocking
- The difference between synchronous and `Async`-suffixed chaining methods
- How to properly handle errors in an asynchronous pipeline

This note is the payoff for [[07 - Future]]'s "what problems does raw Future have?" setup, and connects directly to [[48 - Fork Join Pool]] (its default execution engine).

---

## Why should I learn this?

Recall from [[07 - Future]]: raw `Future` can only be blocked on via `.get()`, cannot be chained, cannot be combined with other futures, and cannot be manually completed. `CompletableFuture`, introduced in Java 8, solves **all** of these limitations, providing a full toolkit for building non-blocking, composable asynchronous pipelines — exactly the kind of code powering modern reactive and async Java systems, including much of Spring WebFlux and reactive microservice architectures.

---

## Beginner Explanation

Recall the "claim ticket" analogy from [[07 - Future]] — you get a ticket, and you can only wait around and check when you're ready to collect. `CompletableFuture` upgrades this: instead of a passive ticket, you get something more like a **subscription service**: "the moment this is ready, automatically do the next thing with it — and if that produces something new, automatically feed it into the next step after that." You never have to sit and wait; you just describe the whole **chain** of what should happen, and the system runs it for you, notifying you (or triggering further actions) only when there's actually something to do.

---

## Real World Analogy

**Assembly Line Analogy**

Imagine an automated assembly line where each station automatically triggers the next the moment its work is done — station 1 finishes machining a part and automatically hands it to station 2 for painting, which automatically hands it to station 3 for packaging — with **no** human standing around checking "is station 1 done yet?" at each step. If two different parts need to be combined at some point (say, a body and an engine, produced on separate lines), the assembly line can wait for **both** to be ready and then automatically combine them.

This is exactly what `CompletableFuture`'s chaining and combining methods let you build — automatic, non-blocking pipelines of dependent asynchronous work.

---

## Technical Explanation

### Creating a `CompletableFuture`

```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    // runs asynchronously, typically on the common ForkJoinPool (see [[48 - Fork Join Pool]])
    return "Hello";
});

CompletableFuture<Void> cfRunnable = CompletableFuture.runAsync(() -> {
    System.out.println("Fire and forget, but still trackable");
});
```

`supplyAsync()` takes a `Supplier<T>` (produces a result); `runAsync()` takes a `Runnable` (no result). Both default to running on the common `ForkJoinPool`, but accept an optional second `Executor` argument if you want a different thread pool.

### Chaining — `thenApply`, `thenAccept`, `thenRun`

```java
CompletableFuture<Integer> result = CompletableFuture
        .supplyAsync(() -> "Hello")
        .thenApply(s -> s + " World")     // transform the result: String -> String
        .thenApply(String::length);        // transform again: String -> Integer

System.out.println(result.get()); // 11
```

- **`thenApply(Function)`**: transforms the result, returns a new `CompletableFuture` with the transformed value.
- **`thenAccept(Consumer)`**: consumes the result (e.g., for side effects like printing), returns `CompletableFuture<Void>`.
- **`thenRun(Runnable)`**: runs some code after completion, ignoring the previous result entirely, returns `CompletableFuture<Void>`.

None of these **block** — they simply register what should happen **next**, once the current stage completes.

### Composing — `thenCompose` (Flattening Nested Futures)

```java
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> "user123");

CompletableFuture<String> ordersFuture = userFuture.thenCompose(userId ->
    CompletableFuture.supplyAsync(() -> fetchOrdersForUser(userId)) // returns ANOTHER CompletableFuture
);
```

`thenCompose()` is used when the transformation function **itself returns a `CompletableFuture`** — without it, you'd end up with a `CompletableFuture<CompletableFuture<String>>` (an awkward nested structure); `thenCompose()` "flattens" this into a single `CompletableFuture<String>`. This is directly analogous to `flatMap()` in Java's `Optional`/`Stream` APIs, if you're familiar with those.

### Combining — `thenCombine` (Two Independent Futures)

```java
CompletableFuture<Integer> priceFuture = CompletableFuture.supplyAsync(() -> 100);
CompletableFuture<Double> taxRateFuture = CompletableFuture.supplyAsync(() -> 0.08);

CompletableFuture<Double> totalFuture = priceFuture.thenCombine(taxRateFuture,
        (price, taxRate) -> price * (1 + taxRate));

System.out.println(totalFuture.get()); // 108.0
```

`thenCombine()` waits for **both** futures to complete (running independently, potentially concurrently), then combines their results using the provided function — directly solving one of raw `Future`'s biggest limitations (see [[07 - Future]]).

### Combining Many — `allOf` and `anyOf`

```java
CompletableFuture<Void> all = CompletableFuture.allOf(future1, future2, future3); // waits for ALL
CompletableFuture<Object> any = CompletableFuture.anyOf(future1, future2, future3); // waits for ANY ONE
```

### Exception Handling

```java
CompletableFuture<Integer> result = CompletableFuture
        .supplyAsync(() -> { throw new RuntimeException("Oops"); })
        .exceptionally(ex -> {
            System.out.println("Handled: " + ex.getMessage());
            return -1; // fallback value
        });

System.out.println(result.get()); // -1
```

- **`exceptionally(Function)`**: handles an exception, providing a fallback value — only invoked if the pipeline failed.
- **`handle(BiFunction)`**: invoked regardless of success or failure, receiving **both** the result (or `null`) and the exception (or `null`) — lets you handle both cases in one place.
- **`whenComplete(BiConsumer)`**: similar to `handle`, but for side effects (like logging), doesn't change the result.

### `Async`-Suffixed Variants — Controlling Which Thread Runs the Next Step

```java
cf.thenApply(x -> transform(x));        // may run on the SAME thread that completed the previous stage
cf.thenApplyAsync(x -> transform(x));   // runs on the DEFAULT executor (common ForkJoinPool)
cf.thenApplyAsync(x -> transform(x), myExecutor); // runs on a SPECIFIC executor you provide
```

This distinction matters for controlling exactly where work happens — the non-`Async` version can execute on whichever thread happens to complete the prior stage (potentially the original calling thread, if the future was already complete when chained), while `Async` variants guarantee execution via a thread pool.

---

## Internal Working

Conceptually, a `CompletableFuture` maintains:

1. Its current completion state (pending, completed successfully with a value, or completed exceptionally).
2. A list of registered "next steps" (callbacks) to run once it completes.

When `.complete()` (or the async computation) finishes:

1. The future's internal state is updated with the result (or exception).
2. Every registered callback is triggered — each producing its **own** `CompletableFuture`, which itself may have further registered callbacks, forming a chain.
3. This cascades through the entire chain automatically, with no thread needing to block and poll — each stage's completion directly triggers the next.

---

## Memory Diagram

```
   supplyAsync(task)  --creates-->  CompletableFuture<A> (pending)
        |
        | .thenApply(f)
        v
   CompletableFuture<B> (pending, waiting on the first)
        |
        | .thenCombine(otherFuture, g)
        v
   CompletableFuture<C> (pending, waiting on BOTH B and otherFuture)

   When the original task completes:
   -> triggers thenApply's callback -> completes B
   -> B's completion (once otherFuture ALSO completes) triggers thenCombine's callback -> completes C
```

---

## Execution Flow

```
supplyAsync(supplier)  -- runs on ForkJoinPool, non-blocking
        |
        v
.thenApply(fn) registered -- NOT executed yet, just registered
        |
   (async task eventually completes)
        |
        v
Registered callback (thenApply's fn) fires automatically
        |
        v
Result propagates to next stage in the chain
        |
   (repeats for however many chained stages exist)
        |
        v
Final .get() (if called) blocks only if not yet complete;
otherwise returns immediately
```

---

## Code Example

```java
import java.util.concurrent.CompletableFuture;
import java.util.List;

public class CompletableFutureDemo {

    static String fetchUser(String id) {
        sleep(100);
        return "User-" + id;
    }

    static int fetchOrderCount(String user) {
        sleep(100);
        return user.hashCode() % 10;
    }

    static void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }

    public static void main(String[] args) throws Exception {

        long start = System.currentTimeMillis();

        // Chained pipeline: fetch user -> fetch their order count -> format a message
        CompletableFuture<String> pipeline = CompletableFuture
                .supplyAsync(() -> fetchUser("42"))
                .thenApply(user -> "Processing " + user)
                .thenApply(String::toUpperCase);

        System.out.println(pipeline.get());

        // Combining two independent async calls
        CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> fetchUser("7"));
        CompletableFuture<Integer> orderCountFuture = CompletableFuture.supplyAsync(() -> fetchOrderCount("User-7"));

        CompletableFuture<String> combined = userFuture.thenCombine(orderCountFuture,
                (user, count) -> user + " has " + count + " orders");
        System.out.println(combined.get());
        System.out.println("Combined call took: " + (System.currentTimeMillis() - start) + "ms (should be ~100-200ms, NOT 400ms)");

        // Error handling
        CompletableFuture<Integer> risky = CompletableFuture
                .supplyAsync(() -> { throw new RuntimeException("simulated failure"); })
                .exceptionally(ex -> {
                    System.out.println("Recovered from: " + ex.getMessage());
                    return -1;
                });
        System.out.println("Risky result: " + risky.get());

        // allOf -- waiting for many
        List<CompletableFuture<String>> many = List.of(
                CompletableFuture.supplyAsync(() -> fetchUser("1")),
                CompletableFuture.supplyAsync(() -> fetchUser("2")),
                CompletableFuture.supplyAsync(() -> fetchUser("3"))
        );
        CompletableFuture.allOf(many.toArray(new CompletableFuture[0])).join();
        many.forEach(f -> System.out.println("Fetched: " + f.join()));
    }
}
```

### Line-by-line explanation

1. The first pipeline demonstrates simple chaining: fetch → transform → transform again, all non-blocking until the final `.get()`.
2. `userFuture` and `orderCountFuture` run **concurrently** (both `supplyAsync` calls start independently), and `thenCombine` waits for both — the total time should reflect roughly the **maximum** of the two (~100ms), not their sum (~200ms), demonstrating genuine concurrency, not the "one submit, immediately block" anti-pattern warned about in [[06 - Callable]].
3. `exceptionally()` catches the simulated failure and provides a fallback value (`-1`), demonstrating clean, functional-style error handling.
4. `allOf()` waits for all three independent fetches to complete before proceeding to print their results.

---

## Dry Run

```
PROCESSING USER-42
User-7 has 3 orders
Combined call took: 105ms (should be ~100-200ms, NOT 400ms)
Recovered from: simulated failure
Risky result: -1
Fetched: User-1
Fetched: User-2
Fetched: User-3
```

---

## Common Mistakes

1. **Calling `.get()` or `.join()` immediately after each stage, defeating the purpose of chaining.**
    
    - This forces synchronous, blocking execution — the whole value of `CompletableFuture` comes from chaining and letting stages run asynchronously without blocking in between.
2. **Confusing `thenApply` with `thenCompose`.**
    
    - Use `thenApply` when your function returns a plain value; use `thenCompose` when your function itself returns another `CompletableFuture` — using `thenApply` in the latter case produces an awkward, nested `CompletableFuture<CompletableFuture<T>>`.
3. **Not handling exceptions anywhere in a long pipeline.**
    
    - An unhandled exception in any stage propagates through the entire chain, and calling `.get()` on the final stage will throw `ExecutionException` wrapping it — always include `.exceptionally()` or `.handle()` where appropriate.
4. **Assuming the common `ForkJoinPool` is always the right executor.**
    
    - For CPU-bound work, it's fine; for I/O-bound work (e.g., blocking network calls inside `supplyAsync`), using the common pool can starve it of threads needed for other parallel stream/CompletableFuture work elsewhere in the JVM — provide a dedicated `Executor` for I/O-bound async work instead.

---

## Production Usage

- **Microservice orchestration**: calling multiple downstream services in parallel and combining their results (exactly like this note's `thenCombine` example) is one of the most common real-world `CompletableFuture` use cases.
- **Reactive-style request handling**: many modern web frameworks (including parts of Spring) support returning a `CompletableFuture<ResponseEntity<T>>` from a controller method, letting the framework handle the response asynchronously once it's ready.
- **Retry and timeout composition**: `CompletableFuture` combined with `orTimeout()`/`completeOnTimeout()` (Java 9+) provides clean, composable timeout handling for async pipelines.
- **Parallel aggregation from multiple data sources**: fetching from several caches/databases/services concurrently and combining the results into a single response, without manually managing raw threads or `Future.get()` blocking.

---

## Interview Questions

**Beginner:**

1. What is `CompletableFuture`, and what limitation of `Future` does it address?
2. What does `thenApply()` do?
3. Does calling `thenApply()` block the calling thread?

**Intermediate:** 4. What's the difference between `thenApply()` and `thenCompose()`? 5. What does `thenCombine()` do, and when would you use it? 6. What's the difference between `exceptionally()` and `handle()`?

**Advanced:** 7. What's the difference between `thenApply()` and `thenApplyAsync()`, in terms of which thread executes the callback? 8. Explain how `allOf()` differs from manually calling `.get()` on each future in a list sequentially.

**Production-Level:** 9. Describe a real production scenario where you'd use `thenCombine()` or `allOf()` to aggregate results from multiple services. 10. Why might you provide a custom `Executor` to `supplyAsync()` instead of relying on the default common pool?

---

## Follow-up Questions

- Does `CompletableFuture` support manual completion, unlike raw `Future`? _(Yes — via `complete(value)`, `completeExceptionally(ex)`, directly addressing one of raw `Future`'s key limitations noted in [[07 - Future]].)_
- Is `CompletableFuture` a subtype of `Future`? _(Yes — it implements `Future<T>`, so all the standard `Future` methods like `.get()`, `.isDone()`, `.cancel()` are also available, in addition to its extensive chaining/composition API.)_
- Does chaining with `thenApply()` guarantee the callback runs on a NEW thread? _(Not necessarily — non-`Async` variants may run on whichever thread completed the previous stage, potentially even the original calling thread if the future was already complete; use the `Async` variants for a guaranteed thread-pool-based execution.)_

---

## Best Practices

- Chain operations using `thenApply`/`thenCompose`/`thenCombine` rather than blocking with `.get()` between steps.
- Use `thenCompose` (not `thenApply`) whenever your transformation function itself returns a `CompletableFuture`.
- Always include exception handling (`exceptionally`/`handle`) somewhere in any non-trivial pipeline.
- Provide a dedicated `Executor` for I/O-bound async work, reserving the common `ForkJoinPool` for genuinely CPU-bound work.

---

## Performance Notes

- Chaining avoids the thread-blocking cost of repeatedly calling `.get()`, letting threads be freed up for other work between stages rather than sitting idle waiting.
- `thenCombine`/`allOf` allow genuinely concurrent execution of independent operations, providing real speedup over sequential blocking calls — as demonstrated in this note's dry run (~100-200ms combined, not ~400ms sequential).
- Using the shared common pool for blocking I/O work can create contention with other parts of the application also relying on that same pool (parallel streams, other `CompletableFuture` chains) — a real, sometimes-overlooked production performance risk.

---

## Comparison Table

|Aspect|`Future` (see [[07 - Future]])|`CompletableFuture`|
|---|---|---|
|Blocking retrieval|Yes (`get()`)|Optional — supports non-blocking chaining|
|Chaining multiple async steps|Not supported directly|`thenApply`, `thenCompose`, etc.|
|Combining multiple futures|Manual, verbose|`thenCombine`, `allOf`, `anyOf`|
|Manual completion|Not supported|`complete()`, `completeExceptionally()`|
|Exception handling|Via `ExecutionException` on `get()`|`exceptionally()`, `handle()`|

---

## Summary

- `CompletableFuture`, introduced in Java 8, solves raw `Future`'s major limitations: no chaining, no combining, no manual completion, awkward exception handling.
- `thenApply`/`thenAccept`/`thenRun` chain sequential steps; `thenCompose` flattens nested futures; `thenCombine`/`allOf`/`anyOf` combine multiple independent futures.
- `exceptionally()` and `handle()` provide clean, functional-style error handling within the pipeline.
- `Async`-suffixed methods let you control exactly which executor runs each stage.
- This is the foundation of most modern asynchronous, non-blocking Java code, and defaults to running on the common `ForkJoinPool` (see [[48 - Fork Join Pool]]).

---

## Cheat Sheet

|Method|Purpose|
|---|---|
|`supplyAsync(supplier)`|Start an async computation with a result|
|`runAsync(runnable)`|Start an async computation with no result|
|`thenApply(fn)`|Transform the result|
|`thenCompose(fn)`|Chain to another `CompletableFuture`-returning step|
|`thenCombine(other, fn)`|Combine with another independent future|
|`allOf(...)` / `anyOf(...)`|Wait for all / any of several futures|
|`exceptionally(fn)`|Provide a fallback on failure|
|`handle(fn)`|Handle both success and failure in one place|

---

## Keywords

CompletableFuture, thenApply, thenCompose, thenCombine, allOf, anyOf, exceptionally, supplyAsync, Non-Blocking

---

## Related Notes

[[07 - Future]] [[48 - Fork Join Pool]] [[46 - ExecutorService]] [[77 - Async in Spring]]

---

## Practice Exercises

**Easy:**

1. Create a `CompletableFuture` using `supplyAsync()`, chain two `thenApply()` transformations, and print the final result.
2. Deliberately throw an exception inside `supplyAsync()` and handle it with `exceptionally()`.

**Medium:** 3. Reproduce this note's `thenCombine()` example, timing it to confirm the two async calls genuinely ran concurrently rather than sequentially. 4. Use `thenCompose()` to chain two dependent async calls (where the second call's input depends on the first call's async result).

**Hard:** 5. Build a small "aggregate from 3 services" scenario using `allOf()`, where each service is simulated with a random delay, and confirm total time is close to the slowest single service, not the sum of all three.

---

## Mini Project

**Project: Async Order Processing Pipeline**

Build a small console application that simulates an e-commerce order processing pipeline:

1. `validateOrder()` → `checkInventory()` → `chargePayment()` → `sendConfirmation()`, each as a separate async stage using `thenCompose()`/`thenApply()`.
2. Simultaneously, fetch shipping estimate and tax calculation using `thenCombine()`, since they can run independently of the main sequential chain.
3. Add proper exception handling at each critical stage (e.g., payment failure should short-circuit the pipeline with a clear error, not silently continue).
4. Print a full trace of the pipeline's execution with timestamps, confirming stages that can run concurrently do so.

**Goal:** Build a realistic, multi-stage asynchronous business pipeline — directly applying chaining, composition, combination, and error handling together in one coherent, production-flavored scenario.

---

## Common Interview Trap

**Trap Question:** _"If I have two independent `CompletableFuture` calls and want to combine their results, is calling `.get()` on the first, then `.get()` on the second, then combining them manually functionally equivalent to using `thenCombine()`?"_

The trap is assuming these two approaches are equally efficient, just differently styled. The correct, more precise answer: they can be **functionally** equivalent in terms of the final combined result, but they are **not equally efficient**, and the difference matters significantly under real concurrent load. Calling `.get()` on the first future **blocks the calling thread** until it completes, and **only then** does execution even reach the `.get()` call for the second future — if the two operations are independent and could genuinely run concurrently, this sequential-blocking approach loses that concurrency benefit entirely if the second operation's _setup_ (like calling `supplyAsync()` for it) happens only after the first `.get()` returns. Even if both futures were already created/started beforehand, sequential blocking `.get()` calls tie up the calling thread for the combined duration rather than freeing it to do other work. `thenCombine()`, by contrast, is fully non-blocking — it registers a callback that fires automatically once **both** futures complete, without the calling thread ever needing to block and wait synchronously. The functional outcome (the combined result) may look the same, but `thenCombine()` provides genuinely different, superior concurrency and resource-utilization characteristics.

---

## Frequently Asked Questions

**Q: What's the difference between `.get()` and `.join()` on a `CompletableFuture`?** A: `.get()` throws checked `InterruptedException` and `ExecutionException`; `.join()` throws an unchecked `CompletionException` instead, making it more convenient to use inside lambda expressions (like inside `.map()` calls) where checked exceptions are awkward to handle.

**Q: Can a `CompletableFuture` be cancelled, like a regular `Future`?** A: Yes — it implements `Future`, so `.cancel(mayInterruptIfRunning)` is available, though cancellation semantics for already-running async computations follow the same cooperative model discussed in [[07 - Future]] and [[21 - Interrupt()]].

**Q: Does `CompletableFuture` require Java 8 or later?** A: Yes — it was introduced in Java 8, alongside lambda expressions and the Streams API, and its fluent, functional-style chaining API is deliberately designed to pair naturally with lambdas.

**Q: Is `CompletableFuture` used internally by reactive frameworks like Project Reactor or RxJava?** A: Not directly — those frameworks have their own, more comprehensive reactive stream abstractions (`Mono`/`Flux`, `Observable`), but `CompletableFuture` shares conceptual similarities (non-blocking, composable async operations) and can often be converted to/from these reactive types when integrating with such frameworks.

---

## Revision Checklist

- [ ] I understand the key limitations of raw `Future` that `CompletableFuture` solves.
- [ ] I can distinguish `thenApply`, `thenCompose`, and `thenCombine` and know when to use each.
- [ ] I understand `allOf()`/`anyOf()` for combining multiple futures.
- [ ] I know how to handle exceptions using `exceptionally()`/`handle()`.
- [ ] I understand the difference between non-`Async` and `Async`-suffixed chaining methods.

---

## Flashcards

**Q:** What key limitations of `Future` does `CompletableFuture` address? **A:** No chaining, no combining multiple futures, no manual completion, awkward exception handling.

**Q:** When should you use `thenCompose()` instead of `thenApply()`? **A:** When your transformation function itself returns a `CompletableFuture` (to avoid nested futures).

**Q:** What does `thenCombine()` do? **A:** Waits for two independent futures to both complete, then combines their results with a provided function.

**Q:** What does `exceptionally()` do? **A:** Provides a fallback value if the pipeline fails with an exception.

**Q:** What executor do `supplyAsync()`/`runAsync()` use by default? **A:** The common `ForkJoinPool`, unless a specific `Executor` is provided.

---

## References

- Oracle Java Tutorials — Concurrency (CompletableFuture): https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html
- `java.util.concurrent.CompletableFuture` API Documentation
- `java.util.concurrent.ForkJoinPool` API Documentation (default execution engine)