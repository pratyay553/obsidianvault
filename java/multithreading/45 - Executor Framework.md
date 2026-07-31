# Executor Framework

## Objective

By the end of this note, you will understand:

- The design philosophy behind the Executor Framework: separating "what to run" from "how/where it runs"
- The core interface hierarchy: `Executor` → `ExecutorService` → `ScheduledExecutorService`
- Why this abstraction was introduced in Java 5 and what problems it solved
- How this framework ties together everything from [[04 - Thread Creation]] through [[44 - Thread Pool]]

This note sits between the conceptual thread pool foundation ([[44 - Thread Pool]]) and the concrete, most-used API ([[46 - ExecutorService]]) — it's about the **design** that makes the whole system coherent.

---

## Why should I learn this?

Understanding the Executor Framework's interface hierarchy — not just "how to call `submit()`" — is what lets you reason about **why** the API is shaped the way it is, and confidently navigate related classes (`ScheduledExecutorService`, `ThreadPoolExecutor`, custom executors) that all fit into this same coherent design. This is exactly the kind of architectural understanding that separates "I can use `ExecutorService`" from "I understand the Executor Framework."

---

## Beginner Explanation

Before Java 5, if you wanted to run something concurrently, you had exactly one primitive tool: create a raw `Thread`, tell it what to run (a `Runnable`), and call `.start()`. This tightly coupled **"what to run"** with **"how it gets executed"** — every piece of code that wanted concurrency had to also manage threads directly.

The **Executor Framework**, introduced in Java 5, deliberately **separates these two concerns**:

- **What to run**: a `Runnable` or `Callable` — pure task description, no knowledge of _how_ it will be executed.
- **How/where it runs**: an `Executor` — some implementation that decides the actual execution mechanism (a new thread each time, a thread pool, a single background thread, a scheduled timer, etc.), completely hidden from the calling code.

This is a classic example of the **separation of concerns** design principle, applied specifically to concurrency.

---

## Real World Analogy

**Shipping Company Analogy**

Imagine you need to ship a package. Before this framework existed, you'd have to personally drive the package to its destination every single time — tightly coupling "I need this delivered" with "I personally do the driving."

The Executor Framework is like switching to using a **shipping company**: you simply hand over the package (the task) and say "please deliver this" — you don't know or care whether they use a van, a bike courier, or a large truck with multiple packages consolidated together (the specific execution mechanism). Different shipping companies (different `Executor` implementations) might handle your package differently — one might use a single dedicated driver, another a fleet of drivers sharing the workload — but from your perspective as the sender, the interface is identical: hand over the package, trust it gets delivered.

---

## Technical Explanation

### The Interface Hierarchy

```
Executor  (the simplest, root interface)
   |
   +-- ExecutorService  (adds lifecycle management, Future-based results)
          |
          +-- ScheduledExecutorService  (adds delayed/periodic execution)
```

### `Executor` — The Root Interface

```java
public interface Executor {
    void execute(Runnable command);
}
```

This is deliberately minimal — a **single method**, representing the most fundamental concept: "here's something to run; you decide how." This is the foundation everything else builds on.

```java
Executor executor = command -> new Thread(command).start(); // a trivially simple Executor implementation

executor.execute(() -> System.out.println("Running via a custom Executor"));
```

Notice: this simplistic implementation just creates a new thread per task (exactly the anti-pattern warned about in [[44 - Thread Pool]]) — `Executor` itself makes **no** promises about efficiency or reuse; it's purely the abstraction boundary. Real implementations (like `ThreadPoolExecutor`) provide the actual pooling behavior.

### `ExecutorService` — Adds Lifecycle and Results

```java
public interface ExecutorService extends Executor {
    <T> Future<T> submit(Callable<T> task);
    Future<?> submit(Runnable task);
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    // ... and more
}
```

This is the interface you'll interact with **by far** the most in practice — it adds:

- **`submit()`**: returns a `Future` (see [[07 - Future]]), letting you retrieve results and handle exceptions — something the bare `Executor.execute()` cannot do.
- **Lifecycle management**: `shutdown()`, `shutdownNow()`, `awaitTermination()` — controlling when and how the underlying threads are stopped.
- **Batch operations**: `invokeAll()`, `invokeAny()` — submitting multiple tasks at once with coordinated result collection.

Full detail on all of this is in [[46 - ExecutorService]] — this note focuses on **where it fits** in the overall design.

### `ScheduledExecutorService` — Adds Time-Based Execution

```java
public interface ScheduledExecutorService extends ExecutorService {
    ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
    ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
    ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
}
```

This extends `ExecutorService` with the ability to run tasks **after a delay** or **repeatedly at a fixed interval** — covered fully in [[47 - ScheduledExecutorService]].

### How `Executors` (the Factory Class) Fits In

```java
ExecutorService fixedPool = Executors.newFixedThreadPool(4);
ExecutorService cachedPool = Executors.newCachedThreadPool();
ExecutorService singleThread = Executors.newSingleThreadExecutor();
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
```

`Executors` (plural, note the distinct name from `Executor`/`ExecutorService`) is a **factory/utility class**, providing convenient pre-configured `ThreadPoolExecutor` instances (and `ScheduledThreadPoolExecutor` instances) — it's not part of the interface hierarchy itself, but the most common **entry point** for obtaining an `ExecutorService` implementation. Full detail on each factory method's specific configuration is in [[46 - ExecutorService]].

---

## Internal Working

Conceptually, how the pieces connect:

1. Application code defines **what** to run — a `Runnable` or `Callable`, with zero knowledge of threading.
2. Application code obtains an `ExecutorService` (typically via an `Executors` factory method).
3. Application code calls `submit()` (or `execute()`), handing the task to the executor.
4. **Internally**, the specific `ExecutorService` implementation (usually `ThreadPoolExecutor`) decides exactly how to run it — using its internal thread pool and task queue (see [[44 - Thread Pool]]), completely hidden behind the interface.
5. If a result is needed, the caller uses the returned `Future` (see [[07 - Future]]) to retrieve it — again, without needing to know anything about the underlying thread management.

---

## Memory Diagram

```
   Application Code
   -----------------------------
   |  Runnable / Callable task   |  <- "WHAT to run"
   -----------------------------
              |
              v
   Executor / ExecutorService (interface)  <- the ABSTRACTION BOUNDARY
              |
              v
   ThreadPoolExecutor (concrete implementation)  <- "HOW it runs"
   -----------------------------
   |  Worker threads             |
   |  Task queue                 |
   -----------------------------
```

---

## Execution Flow

```
Define task (Runnable/Callable) -- no threading knowledge needed
        |
        v
Obtain an ExecutorService (via Executors factory, or custom)
        |
        v
Call submit(task) or execute(task)
        |
        v
Executor implementation decides HOW to run it
(thread pool, scheduling, etc. -- hidden from caller)
        |
        v
Caller optionally uses the returned Future for results
```

---

## Code Example

```java
import java.util.concurrent.*;

public class ExecutorFrameworkDemo {

    public static void main(String[] args) throws Exception {

        // 1. Bare Executor -- the simplest possible abstraction
        Executor simpleExecutor = Runnable::run; // runs on the CALLING thread (a valid, if trivial, Executor)
        simpleExecutor.execute(() -> System.out.println("Ran via simple Executor on: " + Thread.currentThread().getName()));

        // 2. ExecutorService -- adds Future-based results and lifecycle
        ExecutorService executorService = Executors.newFixedThreadPool(2);
        Future<Integer> future = executorService.submit(() -> {
            Thread.sleep(100);
            return 42;
        });
        System.out.println("Result from ExecutorService: " + future.get());
        executorService.shutdown();

        // 3. ScheduledExecutorService -- adds time-based execution
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        scheduler.schedule(() ->
                System.out.println("Ran via ScheduledExecutorService after a delay"), 500, TimeUnit.MILLISECONDS);
        Thread.sleep(700);
        scheduler.shutdown();
    }
}
```

### Line-by-line explanation

1. `simpleExecutor` demonstrates that `Executor` can be satisfied by something as trivial as `Runnable::run` — running the task on the **calling** thread itself (a legitimate, if unusual, implementation choice — sometimes genuinely used for "synchronous execution in the same interface shape as async code").
2. `executorService.submit(callable)` returns a `Future`, letting the caller retrieve the eventual result (`42`) — a capability `Executor.execute()` alone doesn't provide.
3. `scheduler.schedule(...)` demonstrates the additional time-based capability `ScheduledExecutorService` adds on top of `ExecutorService`.
4. All three interact with fundamentally different **implementations**, but through a consistent, layered interface family.

---

## Dry Run

```
Ran via simple Executor on: main
Result from ExecutorService: 42
[... 500ms delay ...]
Ran via ScheduledExecutorService after a delay
```

---

## Common Mistakes

1. **Confusing `Executor` (interface) with `Executors` (factory class) due to the similar name.**
    
    - `Executor` is the root interface; `Executors` is a utility class full of static factory methods for obtaining pre-configured executor instances — genuinely easy to mix up when reading or writing code quickly.
2. **Not realizing `ExecutorService` extends `Executor`.**
    
    - This means anywhere an `Executor` is expected, an `ExecutorService` can be passed instead — the hierarchy is designed for this kind of substitutability (Liskov substitution, if you know that term from OOP design principles).
3. **Using the bare `Executor` interface type directly in application code when you actually need `submit()`/`Future` capabilities.**
    
    - If you need result retrieval or lifecycle management, declare your variable as `ExecutorService`, not just `Executor` — using the narrower interface type unnecessarily limits what you can do with it.
4. **Assuming all `Executor` implementations behave like a thread pool.**
    
    - As shown in the code example, `Runnable::run` is a perfectly valid `Executor` that doesn't use any additional threads at all — the interface makes no promises about concurrency, only about "something will execute this."

---

## Production Usage

- **Spring Framework's `TaskExecutor`** abstraction is directly inspired by (and often backed by) `java.util.concurrent.Executor` — Spring's dependency injection can wire in different executor implementations without application code needing to change.
- **Nearly all asynchronous processing in modern Java frameworks** is built on this interface hierarchy somewhere beneath the surface, even when a framework provides its own higher-level abstraction on top.
- **Testing**: a synchronous `Executor` (like `Runnable::run`) is sometimes deliberately used in unit tests to make asynchronous code execute predictably and synchronously for easier assertions, without changing the production code's interface usage at all.

---

## Interview Questions

**Beginner:**

1. What is the Executor Framework, and what problem does it solve?
2. What is the relationship between `Executor` and `ExecutorService`?
3. What does `Executors` (the factory class) provide?

**Intermediate:** 4. What capability does `ExecutorService` add beyond the bare `Executor` interface? 5. What capability does `ScheduledExecutorService` add beyond `ExecutorService`? 6. Can you pass an `ExecutorService` anywhere an `Executor` is expected? Why?

**Advanced:** 7. Why is `Runnable::run` considered a valid, if unusual, `Executor` implementation? 8. Explain the design principle (separation of concerns) that motivated splitting "what to run" from "how it runs" in this framework.

**Production-Level:** 9. How does a framework like Spring leverage this abstraction to let applications swap executor implementations without changing business logic? 10. Why might a synchronous `Executor` implementation be deliberately useful in a testing context?

---

## Follow-up Questions

- Is `Executor` a functional interface? _(Yes — it has exactly one abstract method, `execute(Runnable)`, making it usable with lambda expressions, as shown in the code example.)_
- Does `ExecutorService` provide any scheduling capability on its own? _(No — scheduling (`schedule()`, `scheduleAtFixedRate()`, etc.) is specifically added by `ScheduledExecutorService`, one level further up the hierarchy.)_
- Can you write a custom `ExecutorService` implementation from scratch, rather than using `Executors`' factory methods? _(Yes — though this is uncommon in practice, since `ThreadPoolExecutor` (obtained via `Executors` or constructed directly) covers the vast majority of real-world needs; custom implementations are reserved for genuinely specialized requirements.)_

---

## Best Practices

- Declare variables using the **narrowest** interface type that provides the capabilities you actually need (`Executor` if you only need `execute()`, `ExecutorService` if you need `submit()`/lifecycle, `ScheduledExecutorService` if you need timing).
- Understand that `Executors`' factory methods are convenient starting points, but for production systems, constructing `ThreadPoolExecutor` directly (with explicit, deliberate configuration) is often recommended for full control — covered in [[46 - ExecutorService]].
- Leverage the framework's abstraction boundary when designing your own code: accept an `Executor`/`ExecutorService` as a dependency (rather than hardcoding thread creation), making your code more testable and flexible.

---

## Performance Notes

- The Executor Framework's abstraction itself adds negligible overhead — the performance characteristics come entirely from the specific implementation you choose (which thread pool configuration, queue type, etc.), not from the interface layering itself.
- Choosing the right factory method (or custom `ThreadPoolExecutor` configuration) has a far bigger performance impact than anything related to the interface hierarchy itself — this is covered in depth in [[46 - ExecutorService]] and [[72 - Performance Tuning]].

---

## Comparison Table

|Interface|Adds|Introduced|
|---|---|---|
|`Executor`|Basic `execute(Runnable)`|Java 5|
|`ExecutorService`|`submit()`/`Future`, lifecycle management, batch operations|Java 5|
|`ScheduledExecutorService`|Delayed and periodic execution|Java 5|

---

## Summary

- The Executor Framework separates "what to run" (`Runnable`/`Callable`) from "how it runs" (an `Executor` implementation), a deliberate application of the separation-of-concerns design principle.
- The interface hierarchy layers capability: `Executor` (bare execution) → `ExecutorService` (results, lifecycle) → `ScheduledExecutorService` (timing).
- `Executors` is a separate factory/utility class providing convenient pre-configured implementations, not part of the interface hierarchy itself.
- This framework, introduced in Java 5, underlies virtually all modern concurrent Java code, directly or via frameworks built on top of it.

---

## Cheat Sheet

|Type|Key Capability|
|---|---|
|`Executor`|`execute(Runnable)` — bare execution|
|`ExecutorService`|`submit()` returning `Future`, `shutdown()`, batch methods|
|`ScheduledExecutorService`|`schedule()`, `scheduleAtFixedRate()`, `scheduleWithFixedDelay()`|
|`Executors`|Factory class for obtaining pre-configured instances|

---

## Keywords

Executor Framework, Executor, ExecutorService, ScheduledExecutorService, Executors, Separation of Concerns

---

## Related Notes

[[44 - Thread Pool]] [[46 - ExecutorService]] [[47 - ScheduledExecutorService]] [[07 - Future]]

---

## Practice Exercises

**Easy:**

1. Write a custom `Executor` implementation (a lambda) that logs a message before running each task, then delegates to `new Thread(task).start()`.
2. Use `Executors.newFixedThreadPool()` to obtain an `ExecutorService`, submit a `Callable`, and retrieve its result via `Future`.

**Medium:** 3. Write a method that accepts an `Executor` (not `ExecutorService`) as a parameter, and demonstrate it working correctly with both a real thread pool AND the trivial `Runnable::run` executor. 4. Use `Executors.newScheduledThreadPool()` to schedule a task to run after a delay, and a separate task to run repeatedly at a fixed interval.

**Hard:** 5. Design and implement a simple "logging decorator" `ExecutorService` wrapper — a class implementing `ExecutorService` that delegates all calls to a wrapped, real `ExecutorService`, but logs before/after each `submit()` call — demonstrating the power of coding against the interface.

---

## Mini Project

**Project: Pluggable Task Runner**

Build a small application with a `NotificationService` class that accepts an `Executor` via constructor injection (dependency injection), and uses it to send notifications asynchronously:

1. Implement the service so it works correctly regardless of which `Executor` implementation is injected.
2. Test it with three different executors: a real thread pool (`Executors.newFixedThreadPool()`), a single-thread executor, and the trivial synchronous `Runnable::run` executor (useful for deterministic testing).
3. Demonstrate that the exact same `NotificationService` code works correctly with all three, changing only which executor is passed in.

**Goal:** Directly experience the practical power of coding against the `Executor` abstraction — the same design principle that lets frameworks like Spring swap implementations without touching business logic.

---

## Common Interview Trap

**Trap Question:** _"Since `ExecutorService` extends `Executor`, and provides strictly more functionality, should you always declare your variables as `ExecutorService` instead of `Executor`, to keep your options open?"_

The trap is treating "more capability" as always preferable, ignoring API design principles. The correct, more nuanced answer: you should declare your variable type based on **what your code actually needs**, not the maximum available capability — this is a general software design principle (sometimes phrased as "program to the narrowest interface that satisfies your requirements"), not specific to executors. If a method only ever calls `execute()` and never needs `Future`-based results or lifecycle control, declaring its parameter as `Executor` (rather than `ExecutorService`) makes the code's actual requirements clearer, makes it usable with a wider range of implementations (including the trivial `Runnable::run` executor, useful for testing), and reduces unnecessary coupling to capabilities the code doesn't use. Defaulting to the "biggest" interface "just in case" is generally considered poorer design, not better.

---

## Frequently Asked Questions

**Q: Is `Executor` the same as `Thread`?** A: No — `Executor` is an interface representing "something that can run tasks"; a specific `Executor` implementation might use one thread, many threads, the calling thread itself, or even a completely different execution mechanism (like a distributed task system, in principle) — `Thread` is just one possible building block an `Executor` implementation might use internally.

**Q: Does the Executor Framework replace the need to understand raw `Thread` mechanics?** A: No — quite the opposite; understanding raw threads (Sections 1-3 of this vault) is exactly what lets you understand **how** `ThreadPoolExecutor` (the most common `ExecutorService` implementation) works internally, since it's built directly on those same primitives.

**Q: Is `ScheduledExecutorService` the only way to run delayed/periodic tasks in Java?** A: It's the standard, most common way within the core JDK; some frameworks (like Spring) provide their own higher-level scheduling abstractions (e.g., `@Scheduled` annotations) that are often built on top of `ScheduledExecutorService` internally.

**Q: Can an `Executor` implementation reject tasks?** A: The bare `Executor` interface doesn't define a formal rejection mechanism, but `ExecutorService` implementations (specifically `ThreadPoolExecutor`) can be configured with rejection policies for when the pool and queue are both full — covered in [[46 - ExecutorService]].

---

## Revision Checklist

- [ ] I understand the design philosophy: separating "what to run" from "how it runs."
- [ ] I can describe the interface hierarchy: `Executor` → `ExecutorService` → `ScheduledExecutorService`.
- [ ] I understand the distinction between `Executor` (interface) and `Executors` (factory class).
- [ ] I know what capability each layer of the hierarchy adds.
- [ ] I understand why coding against the narrowest sufficient interface is good practice.

---

## Flashcards

**Q:** What is the single method defined by the `Executor` interface? **A:** `void execute(Runnable command)`.

**Q:** What does `ExecutorService` add beyond `Executor`? **A:** `submit()` returning `Future`, lifecycle management (`shutdown()`, etc.), and batch operations.

**Q:** What does `ScheduledExecutorService` add beyond `ExecutorService`? **A:** Delayed and periodic task execution.

**Q:** What is `Executors` (plural)? **A:** A factory/utility class providing convenient static methods for obtaining pre-configured executor instances.

**Q:** Is `Executor` a functional interface? **A:** Yes — it has exactly one abstract method, making it usable with lambda expressions.

---

## References

- Oracle Java Tutorials — Executor Interfaces: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.Executor` API Documentation
- `java.util.concurrent.ExecutorService` API Documentation
- `java.util.concurrent.ScheduledExecutorService` API Documentation