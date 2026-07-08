# Runnable

## Objective

By the end of this note, you will understand:

- What the `Runnable` interface is and why it exists
- How `Runnable` differs from `Callable` (previewed here, detailed in [[06 - Callable]])
- How `Runnable` is used across the entire Java concurrency ecosystem — not just with raw `Thread`
- Why `Runnable` is considered the "universal task contract" in Java concurrency

This note builds directly on [[04 - Thread Creation]].

---

## Why should I learn this?

`Runnable` is arguably the single most foundational interface in all of Java concurrency. It is used by:

- Raw `Thread` objects
- `ExecutorService` (see [[46 - ExecutorService]])
- `ScheduledExecutorService` (see [[47 - ScheduledExecutorService]])
- Spring's `@Async` and `TaskExecutor` abstractions
- Almost every framework that needs to say "run this piece of code, at some point, on some thread"

If you deeply understand `Runnable`, every higher-level concurrency tool in this vault will make more sense, because they are all, in some way, built around the idea of "a task that can be run."

---

## Beginner Explanation

`Runnable` is one of the simplest interfaces in Java. It has exactly **one method**:

```java
public interface Runnable {
    void run();
}
```

That's it. `Runnable` represents **"a piece of work that can be executed."** It doesn't know or care:

- Whether it will run on the main thread, a new thread, or a thread pool
- When it will run
- How many times it might run

It's simply a **container for a block of code** that says: "here's what to do, whenever someone decides to run me."

Think of `Runnable` as a **to-do list item written on a sticky note.** The sticky note itself doesn't do anything — it just describes a task. Someone (a thread) has to actually pick it up and do the work described on it.

---

## Real World Analogy

**Warehouse Analogy**

Imagine a warehouse where orders come in as **packing slips** (this is your `Runnable`). Each packing slip describes exactly what needs to be packed and shipped — but the slip itself doesn't pack anything.

- Any available **warehouse worker** (a thread) can pick up any packing slip (`Runnable`) and execute it (pack that order).
- The same packing slip format works whether you have 1 worker or 50 workers.
- Because the "instructions" (Runnable) are separate from "who does it" (the thread), you can freely add more workers, reuse workers for multiple slips, or process slips in any order — all without changing the packing slip format itself.

This is exactly how `Runnable` decouples the definition of work from the execution of that work.

---

## Technical Explanation

### The Interface Definition

```java
@FunctionalInterface
public interface Runnable {
    void run();
}
```

Key characteristics:

- **Single abstract method** (`run()`) → makes it a **functional interface**, meaning it can be implemented using a **lambda expression**.
- **`void` return type** → a `Runnable` cannot return any result back to the caller.
- **No `throws` clause** → a `Runnable`'s `run()` method **cannot declare checked exceptions**. If your logic can throw a checked exception (like `IOException`), you must catch and handle it inside `run()`, or wrap it in an unchecked exception, or use `Callable` instead (see [[06 - Callable]]).

### Three ways to create a `Runnable`

**1. Anonymous class:**

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running anonymously");
    }
};
```

**2. Named class:**

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Running from a named class");
    }
}
```

**3. Lambda expression (modern, most common):**

```java
Runnable task = () -> System.out.println("Running from a lambda");
```

### Where `Runnable` is accepted throughout the JDK

`Runnable` isn't just for `Thread` — it's a **universal currency** accepted by many concurrency APIs:

|API|Method accepting Runnable|
|---|---|
|`Thread`|`new Thread(Runnable target)`|
|`ExecutorService`|`execute(Runnable command)`, `submit(Runnable task)`|
|`ScheduledExecutorService`|`schedule(Runnable command, long delay, TimeUnit unit)`|
|`Spring TaskExecutor`|`execute(Runnable task)`|

This is why `Runnable` is worth deeply understanding early — it's the "lowest common denominator" task type across almost the entire Java (and Spring) concurrency world.

---

## Internal Working

When a `Runnable` is executed by a thread, here's what happens step by step:

1. A `Runnable` instance is created (holding a reference to your task's logic in its `run()` method).
2. This `Runnable` is handed to something that can execute it — a `Thread`, an `ExecutorService`, etc.
3. When execution begins, whatever mechanism is running it (a thread) calls `runnableInstance.run()` — this is a **normal, synchronous method call** from that thread's perspective.
4. The code inside `run()` executes just like any other method — using the calling thread's stack, the process's shared heap, and so on.
5. When `run()` returns, the task is considered "done" as far as that particular invocation goes. If the `Runnable` is stateless (holds no fields), it can safely be reused and executed again, even concurrently, by multiple threads.

**Important distinction**: calling `.run()` directly on a `Runnable` object (without going through `Thread.start()` or an executor) executes the code on the **current** thread — no concurrency happens unless something explicitly runs it on a separate thread.

---

## Memory Diagram

```
              Runnable is just a "task descriptor" object
      ------------------------------------------------------
      |   Runnable instance (on the heap)                   |
      |     - run() method body (your logic)                |
      ------------------------------------------------------
                  ^                    ^                  ^
                  |                    |                  |
             Thread A             Thread B          Executor Pool
           calls .run()          calls .run()        Worker Thread
                                                       calls .run()

      The SAME Runnable instance can be executed by
      different threads (as long as the logic is
      stateless or properly synchronized).
```

---

## Execution Flow

```
Runnable task defined
        |
        v
Handed to a Thread OR an ExecutorService
        |
        v
Execution mechanism decides WHEN and WHERE to run it
        |
        v
Some thread eventually calls task.run()
        |
        v
Code inside run() executes on that thread
        |
        v
run() returns -> task execution is complete
```

---

## Code Example

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class RunnableDemo {

    public static void main(String[] args) throws InterruptedException {

        // Define a reusable Runnable task
        Runnable printTask = () -> {
            System.out.println("Task executed by: " + Thread.currentThread().getName());
        };

        // 1. Run it directly (NO new thread - just a normal method call)
        printTask.run();

        // 2. Run it on a manually created Thread
        Thread t = new Thread(printTask, "Manual-Thread");
        t.start();
        t.join();

        // 3. Run it via an ExecutorService (thread pool) - see [[46 - ExecutorService]]
        ExecutorService executor = Executors.newFixedThreadPool(2);
        executor.execute(printTask);
        executor.execute(printTask); // same Runnable, submitted twice
        executor.shutdown();
    }
}
```

### Line-by-line explanation

1. `Runnable printTask = () -> {...}` — defines a single, reusable task using a lambda.
2. `printTask.run();` — calls `run()` **directly** on the current thread (`main`). No new thread is created here — this line demonstrates that a `Runnable` by itself does nothing concurrent; it's just a method call.
3. `new Thread(printTask, "Manual-Thread").start()` — this time, the exact same `Runnable` instance is executed on a **new** thread.
4. `executor.execute(printTask)` — the same `Runnable` is submitted to a thread pool, which will run it on one of its worker threads (see [[44 - Thread Pool]], [[46 - ExecutorService]]).
5. Notice that the **same `printTask` object** is reused in three completely different execution contexts — this is only safe because the task itself is stateless (it doesn't read/write any shared mutable field).

---

## Dry Run

Expected (though thread name for the executor may vary) output:

```
Task executed by: main
Task executed by: Manual-Thread
Task executed by: pool-1-thread-1
Task executed by: pool-1-thread-2
```

- Line 1 comes from the **direct `.run()` call** — notice it says `main`, proving no new thread was involved.
- Line 2 comes from the **manually created thread**.
- Lines 3–4 come from the **thread pool**, and their exact order (or even which pool thread number executes which submission) is not strictly guaranteed.

---

## Common Mistakes

1. **Calling `.run()` and expecting it to run on a new thread.**
    
    - As shown above, `.run()` is just a plain method call — it always executes on whichever thread calls it. Only `Thread.start()` or an executor's submission methods actually create/use separate threads.
2. **Putting mutable, shared state inside a `Runnable` without synchronization.**
    
    - If a `Runnable` instance is reused across multiple threads and it has instance fields that get modified, this is a direct path to race conditions (see [[26 - Race Condition]]).
3. **Trying to return a value from `run()`.**
    
    - `Runnable.run()` is `void` — it cannot return anything. Beginners often try to work around this with instance fields (which introduces thread-safety issues) instead of simply using `Callable`, which is designed for this exact purpose (see [[06 - Callable]]).
4. **Trying to throw a checked exception from inside `run()`.**
    
    - This won't compile — `run()`'s signature does not declare any checked exceptions. You must catch and handle them inside the method, or convert them to unchecked exceptions.

---

## Production Usage

- **Spring's `@Async` methods** are internally wrapped as `Runnable` (or `Callable`) tasks and submitted to a configured `TaskExecutor`.
- **`ExecutorService.execute(Runnable)`** is the standard way production code submits fire-and-forget background tasks (e.g., sending an email notification without blocking the main request thread).
- **Scheduled jobs** (e.g., "run this cleanup task every night at 2 AM") are typically defined as `Runnable` tasks submitted to a `ScheduledExecutorService` (see [[47 - ScheduledExecutorService]]).
- **Message queue consumers** (Kafka, RabbitMQ) often wrap message-processing logic in `Runnable` tasks that get submitted to a worker thread pool for parallel processing.

---

## Interview Questions

**Beginner:**

1. What is the `Runnable` interface, and how many methods does it define?
2. Can `Runnable` return a value? Why or why not?
3. What is the difference between calling `.run()` directly and calling `.start()` on a `Thread` wrapping that `Runnable`?

**Intermediate:** 4. Why is `Runnable` considered a functional interface, and what does that let you do in modern Java? 5. Name at least 3 different Java concurrency APIs that accept a `Runnable`. 6. What happens if a `Runnable`'s `run()` method needs to throw a checked exception?

**Advanced:** 7. Is it safe to submit the same `Runnable` instance to multiple threads simultaneously? Under what conditions? 8. How does `ExecutorService.execute(Runnable)` differ from `ExecutorService.submit(Runnable)` in terms of return type and exception handling? _(Full detail in [[46 - ExecutorService]].)_

**Production-Level:** 9. In a Spring Boot application, how is a method annotated with `@Async` typically executed under the hood, in terms of `Runnable`/`Callable` and thread pools? 10. Why might reusing a single `Runnable` instance across a high-throughput thread pool be dangerous if that `Runnable` holds mutable instance state?

---

## Follow-up Questions

- Can a lambda expression implement `Runnable` even though lambdas don't explicitly say `implements Runnable`? _(Yes — the compiler infers this based on the target type expected, e.g., the `Thread` constructor or `executor.execute(...)` parameter type.)_
- If two threads execute the exact same stateless `Runnable` instance at the same time, is that safe? _(Yes, as long as the `run()` method doesn't read/write shared mutable state — pure, stateless logic is inherently safe to run concurrently.)_
- Does creating a `Runnable` object consume any OS-level thread resources? _(No — a `Runnable` is just a plain Java object; OS thread resources are only consumed when it's actually executed by a `Thread.start()` call or a pool's worker thread.)_

---

## Best Practices

- Prefer lambda expressions for simple, short `Runnable` tasks.
- Keep `Runnable` implementations **stateless** whenever possible, especially if they'll be reused across multiple threads or submitted to a shared thread pool.
- Always handle checked exceptions inside `run()` — don't let them silently disappear; log them or wrap them appropriately.
- Prefer submitting `Runnable` tasks to an `ExecutorService` rather than manually creating raw threads for each task (see [[44 - Thread Pool]]).

---

## Performance Notes

- Creating a `Runnable` object itself is extremely cheap — it's just a small object (or, with lambdas, often optimized further by the JVM).
- The real performance cost comes from **how** the `Runnable` is executed — a new raw thread per task is expensive; submitting to a reused thread pool is cheap.
- Stateless `Runnable` instances can be safely reused indefinitely, avoiding unnecessary object allocation in high-throughput systems.

---

## Comparison Table

|Aspect|`Runnable`|`Callable<V>` (see [[06 - Callable]])|
|---|---|---|
|Method|`void run()`|`V call() throws Exception`|
|Can return a value?|No|Yes|
|Can throw checked exceptions?|No|Yes|
|Introduced in|Java 1.0|Java 5|
|Common use|Fire-and-forget tasks|Tasks that need a result or may throw|

---

## Summary

- `Runnable` is a functional interface with a single method: `void run()`.
- It represents "a task that can be executed," completely decoupled from _how_ or _when_ it gets executed.
- It cannot return a value or throw checked exceptions — for that, use `Callable`.
- `Runnable` is accepted by nearly every Java concurrency API (`Thread`, `ExecutorService`, `ScheduledExecutorService`, Spring's `TaskExecutor`), making it the foundational "task currency" of Java concurrency.
- Calling `.run()` directly does **not** create a new thread — only `Thread.start()` or an executor's submission methods do that.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|`Runnable`|Functional interface with `void run()`|
|Return value?|Not supported (use `Callable`)|
|Checked exceptions?|Not supported (use `Callable` or handle internally)|
|Creating one|Anonymous class, named class, or lambda|
|Executing one|`Thread.start()`, `ExecutorService.execute()/submit()`|
|Direct `.run()` call|Runs on the current thread — no concurrency|

---

## Keywords

Runnable, Functional Interface, Lambda, Task, `run()`, ExecutorService, Stateless, Checked Exception

---

## Related Notes

[[04 - Thread Creation]] [[06 - Callable]] [[44 - Thread Pool]] [[46 - ExecutorService]]

---

## Practice Exercises

**Easy:**

1. Create a `Runnable` using a lambda expression that prints "Hello Runnable" and run it directly (not via a thread).
2. Wrap that same `Runnable` in a `Thread`, start it, and observe the difference in which thread name is printed.

**Medium:** 3. Create a single `Runnable` instance and submit it to a `Thread` twice by creating two separate `Thread` objects — confirm both can execute the same task instance. 4. Try writing a `Runnable` that attempts to throw a checked exception (like `IOException`) directly from `run()` without a try-catch, and observe the compiler error.

**Hard:** 5. Create a `Runnable` with a mutable instance field (e.g., a counter) and run it concurrently from 5 different threads without synchronization. Observe incorrect final results — this is a hands-on preview of race conditions tied specifically to shared `Runnable` state.

---

## Mini Project

**Project: Task Notification System (Mini Version)**

Build a small console application that:

1. Defines a `Runnable` representing a "send notification" task (just prints a message with a timestamp).
2. Executes it three different ways: directly via `.run()`, via a manually created `Thread`, and via an `ExecutorService` with 2 worker threads.
3. Logs which thread executed each invocation, to clearly demonstrate the difference between the three execution paths.

**Goal:** Build a concrete mental model of `Runnable` as "the task," fully separate from "the thread" that executes it.

---

## Common Interview Trap

**Trap Question:** _"If `Runnable` can't return a value, how do frameworks like Spring handle `@Async` methods that DO return a value (like a `CompletableFuture<String>`)?"_

The trap here is assuming `Runnable` is the only mechanism at play. In reality, when an `@Async` method needs to return a value, Spring internally uses `Callable` (or wraps the logic to populate a `CompletableFuture` or similar) — not a plain `Runnable`. `Runnable` is strictly for fire-and-forget tasks with no return value; anything that needs a result requires `Callable` or a `Future`-based mechanism (see [[06 - Callable]], [[07 - Future]], [[49 - CompletableFuture]]).

---

## Frequently Asked Questions

**Q: Is `Runnable` still relevant in modern Java, or has it been replaced by newer constructs?** A: Still highly relevant — it remains the standard, lightweight way to represent "a task with no return value," and is used extensively throughout the JDK and frameworks like Spring, even in the most modern Java versions.

**Q: Can a `Runnable` be scheduled to run repeatedly?** A: Not by itself — but `ScheduledExecutorService` (see [[47 - ScheduledExecutorService]]) can repeatedly execute a given `Runnable` at fixed intervals or with fixed delays.

**Q: Is it thread-safe to submit the same `Runnable` instance to a thread pool multiple times?** A: Yes, as long as the `Runnable`'s `run()` method doesn't read/write shared mutable state without proper synchronization. Stateless tasks are inherently safe to reuse.

**Q: Does `Runnable` exist only for use with `Thread`?** A: No — it's used across the entire Java concurrency ecosystem, including `ExecutorService`, `ScheduledExecutorService`, and framework-level abstractions like Spring's `TaskExecutor`.

---

## Revision Checklist

- [ ] I can explain what `Runnable` is and why it has only one method.
- [ ] I understand that `Runnable` cannot return a value or throw checked exceptions.
- [ ] I know at least 3 different APIs that accept a `Runnable`.
- [ ] I understand that calling `.run()` directly does not create a new thread.
- [ ] I understand the risk of using shared mutable state inside a reused `Runnable`.

---

## Flashcards

**Q:** What is the single abstract method in `Runnable`? **A:** `void run()`

**Q:** Can `Runnable` return a value? **A:** No — use `Callable` if you need a return value.

**Q:** Does calling `.run()` directly create a new thread? **A:** No — it just executes the code on the current thread, like a normal method call.

**Q:** Since which Java version has `Runnable` been a functional interface usable with lambdas? **A:** Java 8 (lambda syntax), though the interface itself has existed since Java 1.0.

**Q:** Name two Java concurrency APIs, besides `Thread`, that accept a `Runnable`. **A:** `ExecutorService` (`execute`/`submit`) and `ScheduledExecutorService` (`schedule`).

---

## References

- Oracle Java Tutorials — Defining and Starting a Thread: https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- `java.lang.Runnable` API Documentation
- `java.util.concurrent.ExecutorService` API Documentation