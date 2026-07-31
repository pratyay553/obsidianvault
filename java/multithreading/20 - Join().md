# Join()

## Objective

By the end of this note, you will understand:

- What `Thread.join()` does and exactly what guarantee it provides
- How to use it to coordinate the completion of multiple threads
- The difference between `join()`, `join(millis)`, and no timeout at all
- Why `join()` is one of the most genuinely useful, reliable coordination tools covered so far in this vault

This note builds on [[09 - Thread Lifecycle]] (specifically the `WAITING`/`TIMED_WAITING` states triggered by `join()`) and directly follows [[19 - Yield()]] — a nice contrast, since `join()` is a tool you actually **can** rely on, unlike `yield()`.

---

## Why should I learn this?

Unlike `yield()`, `join()` provides a **real, guaranteed** synchronization mechanism: "wait here until that other thread has completely finished." This is one of the most commonly used coordination tools in basic multithreaded Java code, and it's essential for any scenario where one thread's work genuinely depends on other threads finishing first — like waiting for several parallel computations to complete before combining their results.

---

## Beginner Explanation

Imagine you send 3 friends out to run separate errands, and you want to grab lunch together — but only once **everyone** is back. You wait at the restaurant, and each friend, upon returning, lets you know they're done. Once the **last** friend arrives, you can all finally sit down and eat.

`thread.join()` does exactly this for Java threads: calling `workerThread.join()` from your current thread means **"pause right here, and don't continue until `workerThread` has completely finished running."**

---

## Real World Analogy

**Airport Analogy**

Imagine you're picking up 3 family members arriving on 3 different flights, all landing around the same time. You wait at the arrivals area, and you won't leave the airport until **all three** have come through the gate — even if the first two arrive quickly, you keep waiting until the third, potentially delayed, flight also lands.

This is exactly `join()`'s behavior when called on multiple threads in sequence: you wait for each one, one at a time, and only proceed once every single one you're tracking has finished.

---

## Technical Explanation

### Method Signatures

```java
public final void join() throws InterruptedException
public final void join(long millis) throws InterruptedException
public final void join(long millis, int nanos) throws InterruptedException
```

- `join()` (no arguments) — waits **indefinitely** until the target thread terminates.
- `join(millis)` — waits **up to** the specified duration; if the thread hasn't finished by then, `join()` simply returns anyway (it does **not** throw a `TimeoutException` — this is an important difference from `Future.get(timeout, unit)`, covered in [[07 - Future]]).

### Basic Usage

```java
Thread worker = new Thread(() -> {
    // some work
});
worker.start();

worker.join(); // main thread pauses here until 'worker' finishes
System.out.println("Worker is definitely done now.");
```

### Waiting for Multiple Threads

```java
Thread t1 = new Thread(task1);
Thread t2 = new Thread(task2);
Thread t3 = new Thread(task3);

t1.start();
t2.start();
t3.start();

t1.join();
t2.join();
t3.join();

System.out.println("All three threads have finished.");
```

**Important detail**: calling `.join()` on each thread **sequentially** like this is safe and correct — even though you call `t1.join()` first, by the time that call returns, `t2` and `t3` have likely also been running concurrently in the background the whole time (since they were all started before any `join()` call). You're not accidentally forcing them to run one-at-a-time; you're just checking their completion status in sequence.

### `join(millis)` — Timed Waiting Without a Timeout Exception

```java
worker.join(2000); // wait up to 2 seconds

if (worker.isAlive()) {
    System.out.println("Worker is still running after 2 seconds — proceeding anyway.");
} else {
    System.out.println("Worker finished within 2 seconds.");
}
```

Unlike `Future.get(timeout, unit)`, which throws `TimeoutException` if the result isn't ready in time, `join(millis)` simply **returns silently** if the timeout expires, whether or not the thread has actually finished — you must check `isAlive()` afterward if you need to know which case occurred.

### What `join()` Does Internally

Calling `thread.join()` internally relies on the same `wait()`/`notify()` mechanism covered in [[40 - wait()]] and [[41 - notify()]]: the JVM effectively has the target thread call something equivalent to `notifyAll()` on itself once it terminates, waking up any threads that had called `.join()` on it.

---

## Internal Working

Step-by-step of what happens when `mainThread` calls `worker.join()`:

1. `mainThread` calls `worker.join()`.
2. `mainThread`'s state transitions to `WAITING` (or `TIMED_WAITING`, if a timeout was specified) — see [[09 - Thread Lifecycle]].
3. `mainThread` is suspended, consuming no CPU while waiting.
4. `worker` continues running independently until its `run()` method completes (normally or via an uncaught exception).
5. Once `worker` terminates, the JVM internally signals any threads waiting on it via `join()`.
6. `mainThread` wakes up, transitions back to `RUNNABLE`, and continues executing the code immediately after the `join()` call.

---

## Memory Diagram

```
   Main Thread                     Worker Thread
  -------------                  -----------------
  | worker.join() |  ---wait--->  | Still running... |
  -------------                  -----------------
        |                                |
   State: WAITING                        |
        |                                v
        |                         Worker finishes (run() returns)
        |                                |
        |<---------- signal -------------|
        v
   State: RUNNABLE
   Main thread resumes execution
   immediately after join() call
```

---

## Execution Flow

```
Main thread starts worker thread
        |
        v
Main thread calls worker.join()
        |
        v
Main thread: WAITING (or TIMED_WAITING if timeout given)
        |
        v
Worker thread runs independently, eventually finishes
        |
        v
Main thread wakes up, becomes RUNNABLE
        |
        v
Main thread continues execution AFTER the join() call,
guaranteed that worker has fully completed
```

---

## Code Example

```java
public class JoinDemo {

    public static void main(String[] args) throws InterruptedException {

        long start = System.currentTimeMillis();

        Thread task1 = new Thread(() -> simulateWork("Task-1", 2000), "Task-1");
        Thread task2 = new Thread(() -> simulateWork("Task-2", 3000), "Task-2");
        Thread task3 = new Thread(() -> simulateWork("Task-3", 1000), "Task-3");

        task1.start();
        task2.start();
        task3.start();

        // Wait for ALL three to finish before proceeding
        task1.join();
        task2.join();
        task3.join();

        long totalTime = System.currentTimeMillis() - start;
        System.out.println("All tasks complete. Total time: " + totalTime + "ms");
        System.out.println("(Should be ~3000ms, the longest task, NOT 2000+3000+1000=6000ms)");
    }

    static void simulateWork(String name, long durationMs) {
        try {
            System.out.println(name + " starting...");
            Thread.sleep(durationMs);
            System.out.println(name + " finished after " + durationMs + "ms.");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### Line-by-line explanation

1. Three threads are started, each simulating work of different durations (2s, 3s, 1s).
2. `task1.join(); task2.join(); task3.join();` — the main thread waits for each in turn. Even though `task1.join()` is called first, all three threads have already been running **concurrently** since `.start()` was called on each — so this sequence of `join()` calls does not force sequential execution; it simply ensures the main thread doesn't proceed until every one of them is done.
3. Total time printed should be close to **3000ms** (the longest individual task), **not** 6000ms (the sum of all three) — this demonstrates that the three tasks genuinely ran concurrently, and `join()` was just used to synchronize on their completion.

---

## Dry Run

|Time|Task-1|Task-2|Task-3|Main Thread|
|---|---|---|---|---|
|0ms|Starts|Starts|Starts|Calls `task1.join()`, waits|
|1000ms|Running|Running|Finishes|Still waiting on task1|
|2000ms|Finishes|Running|(done)|`task1.join()` returns, calls `task2.join()`, waits|
|3000ms|(done)|Finishes|(done)|`task2.join()` returns, calls `task3.join()` — returns immediately since task3 already finished|
|3000ms||||Prints total time (~3000ms)|

---

## Common Mistakes

1. **Calling `.join()` immediately after `.start()` for each thread, one at a time, expecting concurrency but accidentally getting sequential execution.**
    
    - E.g., writing `t1.start(); t1.join(); t2.start(); t2.join();` instead of starting all threads first, then joining all of them. The first pattern **forces sequential execution** (defeating the purpose of using multiple threads), while the second pattern correctly allows genuine concurrency. This is one of the most common real beginner mistakes with `join()`.
2. **Forgetting that `join(millis)` does not throw an exception on timeout.**
    
    - Unlike `Future.get(timeout, unit)`, `join(millis)` just returns silently if the timeout expires — you must explicitly check `isAlive()` afterward if you need to distinguish "finished in time" from "timed out."
3. **Not handling `InterruptedException` properly.**
    
    - Like `sleep()` and `wait()`, `join()` throws a checked `InterruptedException` if the waiting thread itself is interrupted while waiting — this must be handled thoughtfully, not silently swallowed (see [[18 - Sleep()]] for the same principle).
4. **Assuming `join()` stops or cancels the target thread.**
    
    - `join()` only **waits** for the thread to finish on its own — it has no power to stop, cancel, or influence the target thread's execution in any way.

---

## Production Usage

- **Parallel computation aggregation**: Splitting a large computation across several threads, then using `join()` on each before combining their partial results — a simple, classic pattern (though `ExecutorService` + `Future`/`invokeAll()` is often preferred for more complex or dynamic scenarios).
- **Application startup coordination**: Waiting for several parallel initialization threads (e.g., warming up different caches or connecting to different external services) to all complete before marking the application as "ready."
- **Simple test code**: Frequently used in unit/integration tests that spin up worker threads and need to wait for their completion before making assertions about the results.
- **Batch job orchestration**: Simple batch processes that fan out work across a fixed, known set of threads and use `join()` to wait for the entire batch to complete before proceeding to the next stage.

---

## Interview Questions

**Beginner:**

1. What does `thread.join()` do?
2. What state does a thread enter when it calls `.join()` on another thread?
3. Does `join()` stop or cancel the target thread?

**Intermediate:** 4. What is the difference between `join()` and `join(millis)`? 5. Does `join(millis)` throw an exception if the timeout expires before the target thread finishes? 6. If you call `t1.start(); t1.join(); t2.start(); t2.join();`, are `t1` and `t2` running concurrently?

**Advanced:** 7. How does `join()` relate internally to `wait()`/`notify()` mechanics? 8. Why is it important to start all threads before calling `.join()` on any of them, if you want genuine concurrency?

**Production-Level:** 9. Describe a real production scenario where you'd use `join()` to coordinate multiple parallel threads before proceeding. 10. When would you prefer `ExecutorService` with `invokeAll()` or `Future`-based coordination over manually calling `join()` on several raw threads?

---

## Follow-up Questions

- Can you call `.join()` on a thread that hasn't been started yet? _(Yes, technically — but since the thread was never started, it's not "alive," so `join()` would return immediately without any actual wait, since there's nothing to wait for.)_
- Can multiple different threads call `.join()` on the same target thread simultaneously? _(Yes — this is safe; all callers will be released once the target thread finishes.)_
- Does calling `.join()` on an already-finished thread block at all? _(No — it returns immediately, since the target thread has already terminated.)_

---

## Best Practices

- Always **start all threads first**, then call `.join()` on each of them afterward, to ensure genuine concurrent execution rather than accidental sequential execution.
- Use `join(millis)` with an explicit `isAlive()` check afterward when you need bounded waiting without an exception-based timeout mechanism.
- Handle `InterruptedException` thoughtfully — don't silently swallow it.
- For more complex coordination needs (dynamic task counts, result aggregation, exception propagation), consider `ExecutorService` with `invokeAll()` or `CompletableFuture` instead of manually managing raw threads and `join()` calls (see [[46 - ExecutorService]], [[49 - CompletableFuture]]).

---

## Performance Notes

- `join()` is efficient — the waiting thread is suspended (similar to `sleep()`/`wait()`) and does not consume CPU cycles while waiting.
- Overuse of sequential `join()` calls in the wrong pattern (starting and joining one thread at a time, as described in Common Mistakes) can accidentally serialize what should be concurrent work, significantly hurting performance without any obvious error message — this is a subtle but impactful mistake to watch for in code review.

---

## Comparison Table

|Aspect|`join()`|`join(millis)`|`Future.get()` (see [[07 - Future]])|
|---|---|---|---|
|Waits indefinitely?|Yes|No (up to `millis`)|Yes (unless timeout overload used)|
|Throws on timeout?|N/A|No — returns silently|Yes (`TimeoutException`, if timeout overload used)|
|Retrieves a result value?|No|No|Yes|
|Works with raw `Thread`?|Yes|Yes|No (needs `ExecutorService`)|

---

## Summary

- `thread.join()` pauses the calling thread until the target thread has completely finished executing.
- `join(millis)` provides a bounded wait, but returns silently on timeout rather than throwing an exception — check `isAlive()` if you need to distinguish the outcome.
- A common mistake is interleaving `.start()` and `.join()` calls per-thread, which accidentally forces sequential execution instead of true concurrency.
- `join()` is a genuinely reliable coordination tool, unlike `yield()` — it provides a real, guaranteed synchronization point.
- For more complex scenarios, `ExecutorService`-based coordination (`invokeAll()`, `Future`, `CompletableFuture`) is often preferred over manual `join()` management.

---

## Cheat Sheet

|Method|Behavior|
|---|---|
|`join()`|Waits indefinitely for the thread to finish|
|`join(millis)`|Waits up to `millis`; returns silently on timeout (check `isAlive()`)|
|Correct pattern|Start ALL threads first, THEN join each one|
|Wrong pattern|`start(); join();` repeated per thread (forces sequential execution)|
|Exception|Throws checked `InterruptedException`|

---

## Keywords

Thread.join, WAITING, TIMED_WAITING, isAlive, InterruptedException, Sequential vs Concurrent, Coordination

---

## Related Notes

[[09 - Thread Lifecycle]] [[19 - Yield()]] [[40 - wait()]] [[41 - notify()]] [[46 - ExecutorService]]

---

## Practice Exercises

**Easy:**

1. Start a single thread doing a 2-second simulated task, call `.join()` on it, and print a message confirming it definitely finished before your next line of code runs.
2. Deliberately write the "wrong pattern" (start/join interleaved for 2 threads) and time it; then fix it to the "right pattern" (start both, then join both) and compare the timing difference.

**Medium:** 3. Use `join(1000)` on a thread doing 3 seconds of work, then check `isAlive()` afterward and print an appropriate message reflecting that it's still running. 4. Write a program with 4 threads doing varying amounts of work, all started first, then joined in sequence — confirm the total time matches the longest individual task, not the sum of all tasks.

**Hard:** 5. Build a small "parallel sum calculator" that splits an array into 4 chunks, processes each chunk's partial sum on a separate thread, uses `join()` to wait for all 4, then combines the partial sums into a final total — comparing timing against a single-threaded version of the same calculation.

---

## Mini Project

**Project: Multi-Stage Startup Coordinator**

Build a small console application that simulates an application startup sequence:

1. Spawns 3 threads representing different initialization tasks: "Connecting to Database" (2s), "Warming Cache" (1.5s), "Loading Configuration" (1s).
2. Starts all 3 concurrently, then uses `join()` to wait for all of them to complete.
3. Only after all 3 have finished, prints "Application is READY" along with the total elapsed time (which should be close to 2 seconds, the longest task).
4. As a bonus, add a 4th thread with a very long duration (10s) and use `join(3000)` combined with `isAlive()` to implement a "startup timeout" that proceeds anyway with a warning if that specific task takes too long.

**Goal:** Build a realistic, hands-on simulation of exactly how `join()` is used in real application startup coordination logic.

---

## Common Interview Trap

**Trap Question:** _"If I write `t1.start(); t1.join(); t2.start(); t2.join();` instead of `t1.start(); t2.start(); t1.join(); t2.join();`, does it make any difference?"_

The trap is assuming both patterns are equivalent, since "both threads eventually finish either way." The critical difference: the first pattern **forces `t2` to wait until `t1` has completely finished** before `t2` even **starts** — because `t1.join()` blocks the main thread, and `t2.start()` doesn't execute until that `join()` call returns. This completely eliminates any concurrency benefit between `t1` and `t2`; they run entirely sequentially, one after another, despite superficially looking like "multithreaded" code. The second pattern correctly starts both threads first, allowing them to run **concurrently**, and only then waits for both to finish. This is a genuinely common, subtle bug that silently destroys the performance benefit of using multiple threads in the first place.

---

## Frequently Asked Questions

**Q: Can `join()` be called on the currently executing thread itself (i.e., can a thread join itself)?** A: Technically it compiles, but it would deadlock — a thread would be waiting forever for itself to finish, which can never happen since it's the one doing the waiting; this should never be done intentionally.

**Q: Does `join()` propagate exceptions from the joined thread back to the calling thread?** A: No — if the target thread throws an uncaught exception, `join()` still returns normally once that thread terminates; it does **not** re-throw that exception to the joining thread. If you need exception propagation, use `Callable`/`Future` instead (see [[06 - Callable]], [[07 - Future]]).

**Q: Is `join()` still useful in modern Java code that primarily uses `ExecutorService`?** A: Less commonly needed for typical task coordination (since `Future`/`invokeAll()` handle this more richly), but still useful for simpler scenarios or when working with raw `Thread` objects directly, including virtual threads (see [[78 - Virtual Threads (Project Loom)]]).

**Q: What does `isAlive()` return for a thread that has never been started?** A: `false` — a thread is only considered "alive" between `.start()` being called and it finishing execution.

---

## Revision Checklist

- [ ] I understand what `join()` guarantees and how it differs from `yield()`.
- [ ] I know the difference between `join()` and `join(millis)`, especially regarding timeout behavior.
- [ ] I understand the critical "start all, then join all" pattern versus the mistaken "start/join per thread" pattern.
- [ ] I know that `join()` does not propagate exceptions from the target thread.
- [ ] I understand how to properly handle `InterruptedException` from `join()`.

---

## Flashcards

**Q:** What does `thread.join()` guarantee? **A:** The calling thread will not proceed past this line until the target thread has completely finished.

**Q:** Does `join(millis)` throw an exception if the timeout expires? **A:** No — it returns silently; use `isAlive()` afterward to check if the thread is still running.

**Q:** What's the correct pattern for starting and joining multiple threads concurrently? **A:** Start all threads first, then call `.join()` on each of them afterward.

**Q:** Does `join()` propagate exceptions thrown inside the target thread? **A:** No.

**Q:** What state does the calling thread enter while waiting in `join()`? **A:** `WAITING` (no timeout) or `TIMED_WAITING` (with a timeout).

---

## References

- Oracle Java Tutorials — Joins: https://docs.oracle.com/javase/tutorial/essential/concurrency/join.html
- `java.lang.Thread` API Documentation (`join`, `isAlive`)