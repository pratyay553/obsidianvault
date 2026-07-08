# FutureTask

## Objective

By the end of this note, you will understand:

- What `FutureTask` is and why it exists as a bridge between `Runnable` and `Future`
- How `FutureTask` lets you run a `Callable` on a raw `Thread`
- How `ExecutorService` uses `FutureTask` internally
- How to manually create and use a `FutureTask` outside of an executor

This note completes the "task and result" trilogy started in [[06 - Callable]] and [[07 - Future]].

---

## Why should I learn this?

You may have noticed a gap in [[06 - Callable]]: `Callable` cannot be passed directly to a `Thread` constructor, because `Thread` only accepts `Runnable`. So how do you run a `Callable` on a plain thread, without an `ExecutorService`?

The answer is `FutureTask` — a concrete class that **implements both `Runnable` and `Future`**, acting as an adapter between the two worlds. Understanding `FutureTask` also demystifies what `ExecutorService.submit()` does internally, which is a common "what's really happening under the hood" interview question.

---

## Beginner Explanation

Recall:
- `Runnable` → something a `Thread` can execute, but with no return value.
- `Callable` → something that returns a value, but a `Thread` can't run it directly.

`FutureTask` solves this by being **both at once**:
- It **implements `Runnable`** → so you *can* pass it to a `Thread` constructor.
- It **implements `Future`** → so you *can* call `.get()` on it to retrieve the result.

Think of `FutureTask` as an **adapter plug**. Imagine you have a device with a European power plug (`Callable`), but your wall socket only accepts American plugs (`Thread` needs `Runnable`). `FutureTask` is the adapter that lets you plug a `Callable` into something that expects a `Runnable`, while still letting you get the "output" (the result) out the other side.

---

## Real World Analogy

**Shopping Mall Analogy**

Imagine a shopping mall's customer service desk. A customer submits a **special order request** (a `Callable` — something that will produce a result, like a custom-made item). The mall gives them a **combined receipt-and-claim-ticket** (`FutureTask`):

- This single piece of paper works as **both** a work order that mall staff can act on (the `Runnable` side — staff can process it), **and** a claim ticket the customer can later present to collect their finished item (the `Future` side — customer calls `.get()`).

`FutureTask` is exactly this "two-in-one" document: staff (a `Thread`) can execute it like a normal task, while the customer (your calling code) can later retrieve the result from that very same object.

---

## Technical Explanation

### Class Declaration

```java
public class FutureTask<V> implements RunnableFuture<V> {
    // RunnableFuture<V> extends both Runnable and Future<V>
}
```

`FutureTask` implements `RunnableFuture<V>`, which is itself defined as:

```java
public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();
}
```

This is the key insight: `RunnableFuture` is simply **`Runnable` + `Future`, combined into one interface.** `FutureTask` is the JDK's concrete, ready-to-use implementation of it.

### Creating a `FutureTask` from a `Callable`

```java
Callable<Integer> callableTask = () -> {
    Thread.sleep(1000);
    return 99;
};

FutureTask<Integer> futureTask = new FutureTask<>(callableTask);

Thread t = new Thread(futureTask); // Now it CAN be passed to a Thread!
t.start();

Integer result = futureTask.get(); // blocks until done, just like a normal Future
```

Notice:
- `futureTask` is passed to `new Thread(...)` — legal, because `FutureTask` implements `Runnable`.
- `futureTask.get()` is called — legal, because `FutureTask` also implements `Future`.

### Creating a `FutureTask` from a `Runnable` (with a fixed result)

`FutureTask` also has a constructor that accepts a `Runnable` plus a predetermined result to return upon completion:

```java
Runnable runnableTask = () -> System.out.println("Doing work...");
FutureTask<String> futureTask = new FutureTask<>(runnableTask, "Task Complete!");

new Thread(futureTask).start();
System.out.println(futureTask.get()); // prints "Task Complete!" once done
```

This is useful when you want `Future`-style tracking (completion status, blocking retrieval) around a plain `Runnable`, without needing a computed return value — you just supply a fixed "success marker" object.

### What `ExecutorService.submit()` Does Internally

When you call `executorService.submit(myCallable)`, the JDK internally does roughly this:

```java
FutureTask<V> futureTask = new FutureTask<>(myCallable);
executorService.execute(futureTask); // execute() only accepts Runnable!
return futureTask; // returned to you as a Future<V>
```

This reveals an elegant design: `ExecutorService.execute(Runnable)` is the **only** low-level execution primitive needed. `submit(Callable)` (and `submit(Runnable)`) are convenience methods built entirely on top of wrapping your task in a `FutureTask` and calling `execute()` with it. This is exactly why `FutureTask` matters conceptually — it's the glue that makes `Future`-based result retrieval possible on top of a `Runnable`-only execution mechanism.

---

## Internal Working

Step-by-step of what happens with a manually created `FutureTask`:

1. You create a `FutureTask<V>`, wrapping either a `Callable<V>` or a `Runnable` + fixed result.
2. You pass this `FutureTask` to a `Thread` (since it's also a `Runnable`), and call `.start()`.
3. The new thread begins execution by calling `futureTask.run()`.
4. Internally, `FutureTask.run()`:
   - Calls the wrapped `Callable.call()` (or `Runnable.run()`, if that constructor was used).
   - Captures the return value (or the exception, if one was thrown).
   - Updates its internal state to `COMPLETED` (or `EXCEPTIONAL` / `CANCELLED`).
5. Meanwhile, if your main thread calls `futureTask.get()`, it blocks until this internal state transition happens, then returns the result or throws `ExecutionException`.

---

## Memory Diagram

```
                     FutureTask<V> object
      --------------------------------------------------
      |  Implements: Runnable  AND  Future<V>            |
      |                                                   |
      |  Wraps: Callable<V> (or Runnable + fixed result)  |
      |                                                   |
      |  Internal state: NEW -> COMPLETING -> COMPLETED   |
      |                          (or CANCELLED/EXCEPTIONAL)|
      --------------------------------------------------
               ^                              ^
               |                              |
       new Thread(futureTask)          futureTask.get()
       .start()  -- Runnable side       -- Future side
```

---

## Execution Flow

```
Create Callable<V> or Runnable
        |
        v
Wrap it in a FutureTask<V>
        |
        v
Pass FutureTask to new Thread(...) and start()
        |
        v
Thread calls futureTask.run() internally
        |
        v
Wrapped Callable.call() (or Runnable.run()) executes
        |
   ------------------------
   |                      |
   v                      v
Success               Exception
(store result)        (store cause)
        |                      |
        --------- futureTask.get() ---------
                        |
                        v
        Returns result, OR throws ExecutionException
```

---

## Code Example

```java
import java.util.concurrent.*;

public class FutureTaskDemo {

    public static void main(String[] args) throws Exception {

        // 1. FutureTask wrapping a Callable
        Callable<Integer> callableTask = () -> {
            System.out.println("Callable running on: " + Thread.currentThread().getName());
            Thread.sleep(1000);
            return 55;
        };

        FutureTask<Integer> futureTask1 = new FutureTask<>(callableTask);
        Thread t1 = new Thread(futureTask1, "FutureTask-Thread-1");
        t1.start();

        System.out.println("Result from Callable-based FutureTask: " + futureTask1.get());

        // 2. FutureTask wrapping a Runnable + fixed result
        Runnable runnableTask = () -> {
            System.out.println("Runnable running on: " + Thread.currentThread().getName());
        };

        FutureTask<String> futureTask2 = new FutureTask<>(runnableTask, "Done!");
        Thread t2 = new Thread(futureTask2, "FutureTask-Thread-2");
        t2.start();

        System.out.println("Result from Runnable-based FutureTask: " + futureTask2.get());
    }
}
```

### Line-by-line explanation

1. `Callable<Integer> callableTask = ...` — a task that returns a computed value.
2. `new FutureTask<>(callableTask)` — wraps the `Callable` in a `FutureTask`, giving us something that is now **both** runnable and future-retrievable.
3. `new Thread(futureTask1, "FutureTask-Thread-1")` — this line is only possible because `FutureTask` implements `Runnable`. This would **not** compile if we tried to pass `callableTask` directly.
4. `futureTask1.get()` — blocks until the wrapped `Callable` finishes, then returns its result (55).
5. The second half shows the alternate constructor: wrapping a plain `Runnable` with a **fixed** result value (`"Done!"`), useful when you want `Future`-style tracking without a computed return value.

---

## Dry Run

| Step | Thread | Action |
|---|---|---|
| 1 | Main | Creates `futureTask1`, starts `t1` |
| 2 | FutureTask-Thread-1 | Runs `callableTask.call()`, sleeps 1s, returns 55 |
| 3 | Main | `futureTask1.get()` blocks ~1s, then returns 55 |
| 4 | Main | Creates `futureTask2`, starts `t2` |
| 5 | FutureTask-Thread-2 | Runs `runnableTask.run()`, prints message |
| 6 | Main | `futureTask2.get()` returns the fixed value `"Done!"` |

---

## Common Mistakes

1. **Trying to pass a `Callable` directly to a `Thread` constructor.**
   - This does not compile — `Thread`'s constructor only accepts `Runnable`. You must wrap the `Callable` in a `FutureTask` first.

2. **Not realizing `ExecutorService.submit()` already does this wrapping for you.**
   - Many developers manually create `FutureTask` objects even when using an `ExecutorService`, not realizing `submit()` already handles this internally — manual `FutureTask` creation is mainly useful when you're **not** using an executor (e.g., working with a raw `Thread`).

3. **Forgetting that a `FutureTask` can only be run once.**
   - Like a regular `Runnable`/`Thread` combination, once a `FutureTask` has completed (or been cancelled), it cannot be reset and re-executed. You'd need to create a new `FutureTask` instance for another run.

4. **Confusing the "Runnable + fixed result" constructor with a computed result.**
   - Beginners sometimes expect `new FutureTask<>(runnable, fixedValue).get()` to return something dynamically computed inside the `Runnable` — it doesn't; it always returns exactly the fixed value you supplied, regardless of what the `Runnable` does internally (since `Runnable.run()` can't produce a return value).

---

## Production Usage

- **Internally used by every `ExecutorService`** — whenever you call `submit()`, the JDK is using `FutureTask` (or an equivalent) behind the scenes, whether you realize it or not.
- **Custom task-scheduling frameworks** sometimes build directly on `FutureTask` when they need `Future`-style result tracking but want more control over thread creation than an `ExecutorService` provides.
- **Testing/mocking frameworks** occasionally use `FutureTask` directly to simulate asynchronous behavior in a controlled, single-threaded test environment.

In most everyday production code, you will rarely construct a `FutureTask` manually — you'll use `ExecutorService.submit()`, which does this for you. But understanding `FutureTask` is essential for truly understanding **how** `submit()` works internally, which frequently comes up in deeper technical interviews.

---

## Interview Questions

**Beginner:**
1. What is `FutureTask`, and what two interfaces does it implement?
2. Why can't you pass a `Callable` directly to a `Thread` constructor?
3. What does `RunnableFuture` represent?

**Intermediate:**
4. How would you run a `Callable` on a plain `Thread` without using an `ExecutorService`?
5. What does the `FutureTask` constructor that accepts a `Runnable` and a fixed value actually return when `.get()` is called?

**Advanced:**
6. What does `ExecutorService.submit(Callable)` do internally, in terms of `FutureTask`?
7. Can a `FutureTask` be reused (re-run) after it completes? Why or why not?

**Production-Level:**
8. In what scenario would you manually create a `FutureTask` instead of relying on `ExecutorService.submit()`?
9. How does understanding `FutureTask` help you reason about custom task scheduling or execution frameworks you might build or maintain?

---

## Follow-up Questions

- Does `FutureTask` itself create a new thread? *(No — `FutureTask` is just a task wrapper; it still needs to be run by something, either a manually created `Thread` or an executor's worker thread.)*
- Is `FutureTask` thread-safe? *(Yes — its internal state transitions and result storage are designed to be safely observed and retrieved from multiple threads.)*
- What happens if you call `.run()` on a `FutureTask` twice? *(The second call typically does nothing meaningful — once a `FutureTask` has completed, it does not re-execute the underlying task; its state is already finalized.)*

---

## Best Practices

- Prefer `ExecutorService.submit()` over manually creating `FutureTask` objects in most production code — it's simpler and less error-prone.
- Reach for manual `FutureTask` construction mainly when you need `Future`-style result tracking **without** the overhead or semantics of a full thread pool.
- Remember that a `FutureTask`, like any `Runnable`/`Thread` pairing, is single-use — create a new instance for each execution.

---

## Performance Notes

- `FutureTask` itself adds negligible overhead — it's a lightweight wrapper object with internal state management.
- The real performance considerations are the same as with `Future` in general: blocking on `.get()` ties up the calling thread, and cancellation is only effective if the wrapped task cooperates with interruption.

---

## Comparison Table

| Aspect | `Runnable` | `Callable` | `FutureTask` |
|---|---|---|---|
| Can pass to `Thread` constructor? | Yes | No | Yes |
| Returns a value? | No | Yes | Yes (via `.get()`) |
| Implements `Future`? | No | No | Yes |
| Typically created manually in production code? | Often | Often | Rarely (executors do it for you) |

---

## Summary

- `FutureTask` implements `RunnableFuture<V>`, which combines `Runnable` and `Future<V>` into a single class.
- It acts as an **adapter**, letting a `Callable` (or a `Runnable` + fixed result) be executed by anything that expects a `Runnable` (like a plain `Thread`), while still supporting `Future`-style result retrieval.
- `ExecutorService.submit()` uses `FutureTask` internally — this is the mechanism that makes `submit()` work despite `execute()` only formally accepting `Runnable`.
- In everyday production code, you rarely construct `FutureTask` manually, but understanding it deeply clarifies how the entire `Callable`/`Future`/`ExecutorService` system fits together.

---

## Cheat Sheet

| Concept | One-liner |
|---|---|
| `FutureTask<V>` | Implements both `Runnable` and `Future<V>` |
| Purpose | Adapter to run a `Callable` on a plain `Thread` |
| Constructor 1 | `new FutureTask<>(callable)` |
| Constructor 2 | `new FutureTask<>(runnable, fixedResult)` |
| Used internally by | `ExecutorService.submit()` |
| Reusable? | No — single execution only |

---

## Keywords

FutureTask, RunnableFuture, Callable, Future, ExecutorService, Adapter Pattern, submit()

---

## Related Notes

[[05 - Runnable]]
[[06 - Callable]]
[[07 - Future]]
[[46 - ExecutorService]]

---

## Practice Exercises

**Easy:**
1. Wrap a `Callable<Integer>` in a `FutureTask`, run it on a manually created `Thread`, and print the result via `.get()`.
2. Wrap a `Runnable` with a fixed result string in a `FutureTask`, run it, and print the fixed result.

**Medium:**
3. Compare timing: create a `Callable`-based task, run it once via a manually created `FutureTask`+`Thread`, and once via `ExecutorService.submit()`. Confirm both approaches produce the same result.
4. Try calling `.get()` on a `FutureTask` **before** starting its thread, and observe that it blocks indefinitely (be ready to stop the program) — this demonstrates why order of operations matters.

**Hard:**
5. Implement a very small custom "mini executor service" (a single background thread that continuously pulls `FutureTask` objects from a shared queue and runs them), to internalize exactly how `ExecutorService.submit()` works under the hood.

---

## Mini Project

**Project: Manual Task Executor**

Build a small console application that:
1. Defines a simple queue (e.g., `BlockingQueue<FutureTask<?>>` — previewed here, fully covered in [[43 - BlockingQueue]]) of tasks to run.
2. Starts a single background `Thread` that continuously polls this queue and runs each `FutureTask` it finds.
3. From the main thread, creates several `Callable` tasks, wraps them in `FutureTask`s, adds them to the queue, and then retrieves and prints each result via `.get()`.

**Goal:** Recreate, in miniature, exactly what a real `ExecutorService` does internally — turning `FutureTask` from an abstract concept into something you've built yourself.

---

## Common Interview Trap

**Trap Question:** *"Since `ExecutorService.execute()` only accepts `Runnable`, how does `submit(Callable)` work at all?"*

The trap here is assuming there must be some hidden, separate execution mechanism for `Callable`. The real answer is elegantly simple: `submit(Callable)` **wraps your `Callable` in a `FutureTask`** (which implements `Runnable`), and then internally calls the exact same `execute(Runnable)` method used everywhere else. There is no separate "Callable execution path" in the JVM — `FutureTask` is the bridge that makes it all work through one unified `Runnable`-based execution mechanism.

---

## Frequently Asked Questions

**Q: Is `FutureTask` a class or an interface?**
A: A concrete class — `java.util.concurrent.FutureTask<V>` — that implements the `RunnableFuture<V>` interface.

**Q: Can I use `FutureTask` without any `ExecutorService` at all?**
A: Yes — you can pass it directly to a plain `Thread` constructor and call `.start()`, since it implements `Runnable`.

**Q: Does `FutureTask` support cancellation like a regular `Future`?**
A: Yes — since it implements `Future`, it supports `cancel()`, `isCancelled()`, and `isDone()` exactly as described in [[07 - Future]].

**Q: If I use the `Runnable` + fixed result constructor, does the fixed result get evaluated lazily or immediately?**
A: The fixed result reference is captured immediately when the `FutureTask` is constructed; it's simply returned by `.get()` once the wrapped `Runnable` finishes running (assuming no exception was thrown).

---

## Revision Checklist

- [ ] I understand that `FutureTask` implements both `Runnable` and `Future`.
- [ ] I can explain why `Callable` needs `FutureTask` to run on a plain `Thread`.
- [ ] I understand that `ExecutorService.submit()` uses `FutureTask` internally.
- [ ] I know the difference between the `Callable`-based and `Runnable`+fixed-result constructors of `FutureTask`.
- [ ] I understand that a `FutureTask`, once completed, cannot be re-run.

---

## Flashcards

**Q:** What two interfaces does `FutureTask` implement (combined into one)?
**A:** `Runnable` and `Future` (via `RunnableFuture`).

**Q:** Why is `FutureTask` needed to run a `Callable` on a raw `Thread`?
**A:** Because `Thread`'s constructor only accepts `Runnable`, and `FutureTask` adapts a `Callable` into something that also implements `Runnable`.

**Q:** What does `ExecutorService.submit(Callable)` do internally?
**A:** It wraps the `Callable` in a `FutureTask` and calls `execute(Runnable)` with it, then returns the `FutureTask` as a `Future`.

**Q:** Can a `FutureTask` be re-executed after completing?
**A:** No — it's single-use, like any other `Runnable`/`Thread` pairing.

**Q:** What does the `FutureTask(Runnable, V result)` constructor return from `.get()`?
**A:** The fixed `result` value you supplied, once the wrapped `Runnable` finishes running successfully.

---

## References

- Oracle Java Tutorials — Executors: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.FutureTask` API Documentation
- `java.util.concurrent.RunnableFuture` API Documentation