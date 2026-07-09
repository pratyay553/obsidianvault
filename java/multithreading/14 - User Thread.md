# User Thread

## Objective

By the end of this note, you will understand:

- What a user (non-daemon) thread is, precisely, as the "default" thread type in Java
- Why user threads are the ones the JVM waits for before exiting
- How to reason about which of your threads should be user threads vs. daemon threads
- How thread daemon-status inheritance works in practice

This note is the direct companion to [[13 - Daemon Thread]]. Read that note first if you haven't already — this note assumes that foundation and focuses on the "default"/"critical work" side of the distinction.

---

## Why should I learn this?

"User thread" is a somewhat unusual term — it's mostly defined **in contrast to** daemon threads, rather than having distinct behavior of its own. But understanding it precisely matters because:

- It's the **default** behavior for every thread you create, so understanding it is really understanding "what happens if I do nothing special."
- It directly explains why your application sometimes refuses to shut down (a very common real-world debugging scenario).
- Interviewers often test whether you understand the relationship correctly — that "user thread" isn't some special extra feature, but simply the absence of the daemon flag.

---

## Beginner Explanation

Every thread you create in Java, by default, is a **user thread** (also sometimes called a "non-daemon thread"). Think of user threads as the "main, essential staff" of your program — the JVM considers their work important enough that it will **wait** for all of them to finish before the whole program is allowed to shut down.

If you do nothing special (never call `setDaemon(true)`), every thread you create — including any spawned by libraries or frameworks, unless they explicitly mark their threads as daemon — is a user thread by default.

---

## Real World Analogy

**Restaurant Analogy (continued from [[13 - Daemon Thread]])**

Recall: user threads are like the restaurant's core staff — the chefs, waiters, and cashier. The restaurant's manager (the JVM) will not lock the doors and go home until every one of these staff members has finished their current task and clocked out.

- If a waiter is still serving a table, the restaurant stays open.
- If the chef is still cooking the last order, the restaurant stays open.
- Only once **every** staff member (every user thread) has finished does the manager finally close up for the night (the JVM exits).

This is fundamentally different from the security guard (a daemon thread, see [[13 - Daemon Thread]]), whose ongoing work does **not** delay closing time.

---

## Technical Explanation

### The Default Behavior

```java
Thread t = new Thread(() -> {
    System.out.println("I am a user thread by default.");
});
System.out.println(t.isDaemon()); // false
```

Every `Thread` object is **non-daemon (a user thread) by default**, unless:

1. It was explicitly marked with `setDaemon(true)` before starting, or
2. It was created **by** a thread that is itself a daemon thread (in which case it inherits daemon status — see the inheritance rule below).

### The JVM Shutdown Rule (from the User Thread's perspective)

As covered in [[13 - Daemon Thread]], the JVM's natural shutdown rule is:

> The JVM will continue running as long as **at least one user (non-daemon) thread** is still alive.

This means:

- If you spawn 10 user threads and your `main()` method returns immediately, the JVM will **still keep running** until all 10 of those threads finish, even though `main()` itself is done.
- This is a common source of confusion for beginners who expect the program to end as soon as `main()` returns.

### Inheritance Rule

When a thread creates another thread, the new thread's default daemon status is **inherited from its parent** (the thread that created it):

```java
Thread parent = new Thread(() -> {
    Thread child = new Thread(() -> System.out.println("Am I daemon? " + Thread.currentThread().isDaemon()));
    child.start(); // child inherits parent's daemon status BY DEFAULT
});
parent.setDaemon(true);
parent.start();
// child will print "Am I daemon? true", because it inherited from its daemon parent
```

If `parent` had **not** been marked as daemon (i.e., it remained a normal user thread), then `child` would also default to being a user thread — this is the far more common, everyday scenario, since most threads you create are user threads unless you deliberately opt into daemon status.

### Framework and Library Threads

Most everyday application threads — including those created by common frameworks and thread pools (`ExecutorService`, Spring's default task executors, etc.) — are **user threads by default**, unless the specific library explicitly configures its threads as daemon. This is precisely why forgetting to shut down an `ExecutorService` is such a common real-world bug: those pooled worker threads are non-daemon by default, so they will keep the JVM alive indefinitely if never properly shut down (see [[46 - ExecutorService]]).

---

## Internal Working

Step-by-step of how user threads affect JVM lifetime:

1. The JVM starts, and the `main` thread — always a user thread — begins executing.
2. As your program runs, it may create additional threads. Each new thread defaults to user (non-daemon) status, unless explicitly configured otherwise or inherited from a daemon parent.
3. As threads complete their work and terminate, the JVM continuously checks: "is there still at least one live user thread?"
4. As long as the answer is "yes," the JVM keeps running — even if `main()` itself has already returned.
5. Only once the answer becomes "no" (every user thread has finished) does the JVM proceed to natural shutdown, at which point any remaining daemon threads are abruptly terminated (see [[13 - Daemon Thread]]).

---

## Memory Diagram

```
        JVM Process
   ---------------------------------
   |  main() [User Thread]          |  <- finishes early
   |  Worker-1 [User Thread]        |  <- still running
   |  Worker-2 [User Thread]        |  <- still running
   |  Metrics-Reporter [Daemon]     |  <- still running
   ---------------------------------

   main() finishes -> JVM still alive (Worker-1, Worker-2 remain)
   Worker-1 finishes -> JVM still alive (Worker-2 remains)
   Worker-2 finishes -> JVM checks: any user threads left? NO

                    JVM EXITS
   Metrics-Reporter (daemon) is abruptly terminated,
   regardless of main()'s return long ago.
```

---

## Execution Flow

```
main() (a user thread) starts
        |
        v
main() creates Worker-1, Worker-2 (both default to user threads)
        |
        v
main() creates Metrics-Reporter, explicitly sets daemon=true
        |
        v
main() finishes and returns
        |
        v
JVM checks: are Worker-1 or Worker-2 (user threads) still alive?
        |
       YES (at least one is)
        |
        v
JVM keeps running, even though main() has already returned
        |
        v
Eventually, Worker-1 and Worker-2 both finish
        |
        v
JVM checks again: any user threads left? NO
        |
        v
JVM exits; Metrics-Reporter (daemon) is cut off
```

---

## Code Example

```java
public class UserThreadDemo {

    public static void main(String[] args) {

        System.out.println("Is main() a daemon thread? " + Thread.currentThread().isDaemon()); // false

        Thread worker = new Thread(() -> {
            System.out.println("Worker (user thread) starting long task...");
            try {
                Thread.sleep(3000); // simulate 3 seconds of important work
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println("Worker finished its important task.");
        }, "Important-Worker");

        // NOT setting daemon -> defaults to user thread (non-daemon)
        System.out.println("Is worker a daemon thread? " + worker.isDaemon()); // false

        worker.start();

        System.out.println("main() is finishing now, but the JVM will WAIT for 'worker' to finish.");
        // main() returns here, but the JVM does NOT exit yet, because 'worker' (a user thread) is still alive.
    }
}
```

### Line-by-line explanation

1. `Thread.currentThread().isDaemon()` inside `main()` — prints `false`, confirming the main thread is always a user thread.
2. `worker` is created **without** calling `setDaemon(true)` — so it defaults to being a user thread as well.
3. Even though `main()` prints its final message and returns almost immediately, the JVM process **does not exit** — it waits a full 3 seconds for `worker` to finish its `Thread.sleep(3000)` and print "Worker finished its important task." **before** the process actually terminates.
4. This directly demonstrates the core rule: **the JVM waits for all user threads, not just the completion of `main()`.**

---

## Dry Run

|Time|Event|
|---|---|
|0.0s|`main()` prints daemon status checks, starts `worker`|
|0.0s|`main()` prints "finishing now" message and returns|
|0.0s → 3.0s|JVM stays alive because `worker` (a user thread) is still running|
|3.0s|`worker` finishes, prints "Worker finished its important task."|
|3.0s|JVM checks: any user threads left? No → JVM exits|

---

## Common Mistakes

1. **Assuming the program ends as soon as `main()` returns.**
    
    - This is one of the most common beginner surprises: if you've spawned any user threads that are still running, the JVM process stays alive until they **all** finish, regardless of `main()`'s completion.
2. **Not realizing thread pool worker threads are user threads by default.**
    
    - This leads directly to the classic "why won't my program exit?" bug — an `ExecutorService` that's never explicitly shut down keeps its (non-daemon) worker threads alive indefinitely, preventing JVM exit (see [[46 - ExecutorService]]).
3. **Confusing "user thread" with "thread created by the end user of the application" (a UX-related meaning).**
    
    - In this JVM/concurrency context, "user thread" has nothing to do with an actual human user — it's simply the technical term for "non-daemon thread." This naming can genuinely confuse beginners.
4. **Assuming all threads created by a library or framework are automatically user threads without checking.**
    
    - While this is the default, some frameworks (especially those managing background/scheduled tasks) deliberately create daemon threads — always check documentation or test behavior if you're unsure, especially when debugging shutdown issues.

---

## Production Usage

- **Core request-processing threads** in servers (e.g., Tomcat's worker threads handling HTTP requests) are user threads — a server should complete an in-flight request before shutting that particular thread down, not abruptly cut it off.
- **Application startup logic** that spawns essential initialization threads (e.g., warming up caches before the app is considered "ready") typically uses user threads, ensuring the JVM doesn't exit prematurely mid-initialization.
- **Graceful shutdown patterns**: many production systems combine user threads (for critical work) with explicit shutdown hooks (`Runtime.addShutdownHook()`) to ensure clean, orderly termination rather than relying solely on the natural "wait for user threads" JVM behavior.
- **Common operational incident**: a batch job or microservice that "hangs" and never exits after apparently finishing its work is very often caused by a forgotten, un-shutdown `ExecutorService` still holding live (user) worker threads.

---

## Interview Questions

**Beginner:**

1. What is a user thread, and how does it differ from a daemon thread?
2. What is the default daemon status of a newly created thread?
3. Does the JVM exit as soon as `main()` returns?

**Intermediate:** 4. If you create 3 user threads and your `main()` method finishes immediately, when does the JVM actually exit? 5. What's the default daemon status of threads created inside an `ExecutorService`? 6. What happens to a user thread's daemon status if it's created by a thread that is itself a daemon thread?

**Advanced:** 7. Explain the exact JVM rule that determines when a Java process naturally exits, in terms of user vs. daemon threads. 8. Why is "user thread" primarily a term that only makes sense in contrast to "daemon thread," rather than having independent defining behavior?

**Production-Level:** 9. Describe a real production incident scenario caused by forgetting that thread pool threads default to user (non-daemon) status. 10. How would you design a service's shutdown sequence to reliably ensure all critical user threads complete their work before the process exits?

---

## Follow-up Questions

- If `main()` creates a user thread and then throws an unhandled exception itself, does the JVM exit immediately? _(No — `main()`'s own thread terminating due to an exception doesn't force JVM exit if other user threads are still running; the JVM still waits for those remaining user threads.)_
- Can you convert a user thread into a daemon thread while it's already running? _(No — daemon status can only be set before `.start()`, exactly as with daemon threads themselves; see [[13 - Daemon Thread]].)_
- Is it possible for a Java program to run forever, unintentionally, because of a forgotten user thread? _(Yes — this is a very real, common issue, typically caused by an un-shutdown `ExecutorService` or a background loop thread that was never intended to run indefinitely.)_

---

## Best Practices

- Use user threads (the default) for any work that must reliably complete before your application is considered "done" — this is almost always the right choice for core business logic.
- Always explicitly call `executorService.shutdown()` (or `shutdownNow()`) when you're finished with a thread pool, to avoid unintentionally keeping the JVM alive.
- When designing background/support threads (metrics, logging, cleanup), deliberately decide: does this **need** to complete before shutdown (use a user thread, likely paired with a shutdown hook for graceful handling), or is it purely supportive (use a daemon thread, see [[13 - Daemon Thread]])?
- Don't assume a program will exit immediately after `main()` returns — always account for any live user threads you've spawned.

---

## Performance Notes

- There is no direct performance difference between user and daemon threads during execution — the distinction only affects JVM shutdown behavior, not runtime performance.
- However, **unintentionally long-lived user threads** (e.g., a forgotten thread pool) can be an operational/resource concern — a process that never exits when expected can accumulate resource usage or block deployment/restart automation in production environments.

---

## Comparison Table

|Aspect|User Thread (Non-Daemon)|Daemon Thread|
|---|---|---|
|Default for new threads?|Yes|No (must opt in)|
|JVM waits for it before exit?|Yes|No|
|Appropriate for critical work?|Yes|No|
|Appropriate for background/support work?|Sometimes (if it must still complete)|Yes, typically|
|Common real-world default (e.g., `ExecutorService`)|Yes|No|

---

## Summary

- A **user thread** (non-daemon thread) is the default type for every new `Thread` object in Java.
- The JVM will **not** exit naturally as long as at least one user thread is still alive — `main()` returning does not, by itself, end the program.
- Threads created by a daemon thread inherit daemon status by default; threads created by a user thread inherit user (non-daemon) status by default.
- Thread pools (`ExecutorService`) default to non-daemon worker threads, which is a very common source of "why won't my JVM exit?" bugs when the pool is never explicitly shut down.
- Use user threads for essential work; reserve daemon threads (see [[13 - Daemon Thread]]) for background/support tasks that shouldn't block shutdown.

---

## Cheat Sheet

|Rule|Detail|
|---|---|
|Default status|Every new thread is a user thread, unless explicitly set to daemon or inherited from a daemon parent|
|JVM exit condition|Only after ALL user threads have finished|
|`main()` returning|Does NOT by itself end the program if user threads are still alive|
|`ExecutorService` threads|User threads by default — must explicitly `shutdown()`|
|Inheritance|New threads inherit their creator's daemon status by default|

---

## Keywords

User Thread, Non-Daemon Thread, JVM Shutdown, Default Thread Behavior, ExecutorService, Inheritance, Graceful Shutdown

---

## Related Notes

[[13 - Daemon Thread]] [[09 - Thread Lifecycle]] [[46 - ExecutorService]] [[61 - InheritableThreadLocal]]

---

## Practice Exercises

**Easy:**

1. Create a thread without setting any daemon status, and confirm via `isDaemon()` that it defaults to `false`.
2. Create a user thread with a 3-second sleep, let `main()` finish immediately, and observe that the program doesn't actually exit until after those 3 seconds.

**Medium:** 3. Create an `ExecutorService`, submit a task, and let `main()` finish without calling `shutdown()`. Observe that the program hangs indefinitely (be ready to stop it manually), then fix it by adding `executorService.shutdown()`. 4. Create a user thread that itself creates a child thread — confirm the child also defaults to being a user thread.

**Hard:** 5. Build a small program with a mix of 2 user threads and 2 daemon threads, all doing work of varying durations, and add logging that prints exactly when the JVM exits relative to each thread's completion — confirm the JVM exit timing matches the rule described in this note.

---

## Mini Project

**Project: Graceful Shutdown Simulator**

Build a small console application that:

1. Simulates "processing a batch of critical orders" using 3 user threads, each taking a different amount of time (1s, 2s, 3s).
2. Simulates a background "system health monitor" as a daemon thread, printing a status message every 500ms.
3. Lets `main()` finish immediately after starting all threads.
4. Confirms (via printed output/timestamps) that the JVM correctly waits for all 3 order-processing user threads to finish, while the daemon health monitor is cut off the moment the last user thread completes.

**Goal:** Build a concrete, timed demonstration of the exact JVM shutdown rule — reinforcing the practical difference between user and daemon threads in a realistic scenario.

---

## Common Interview Trap

**Trap Question:** _"If `main()` throws an unhandled exception and crashes immediately, does the JVM exit right away, even if other user threads are still running?"_

The trap is assuming that `main()` crashing is special and forces immediate JVM exit. In reality, `main()`'s thread terminating (whether normally or via an uncaught exception) is treated **the same as any other user thread finishing** — the JVM still checks whether **any** user threads remain alive. If other user threads are still running, the JVM continues to wait for them, exactly as it would if `main()` had simply returned normally. `main()` has no special "override" power to force immediate shutdown — only `System.exit()` (or a fatal JVM-level crash) does that.

---

## Frequently Asked Questions

**Q: Is the `main` thread ever a daemon thread?** A: No — the JVM always starts `main()` as a user (non-daemon) thread; this cannot be changed.

**Q: If I never explicitly create any additional threads, when does my program exit?** A: As soon as `main()` finishes, since `main()` itself is the only (user) thread, and once it terminates, no user threads remain alive.

**Q: Does calling `Thread.sleep()` inside a user thread count as "still alive" for JVM shutdown purposes?** A: Yes — a sleeping thread is still alive (in the `TIMED_WAITING` state, see [[09 - Thread Lifecycle]]); the JVM will wait for it to wake up and finish, just like any other live user thread.

**Q: Can I check, from within a running application, how many user threads are currently alive?** A: Yes — using `Thread.activeCount()` for a rough count, or more precisely via `ThreadMXBean` (see [[10 - Thread States]]) combined with checking each thread's `isDaemon()` status.

---

## Revision Checklist

- [ ] I understand that user threads are the default type for every new thread.
- [ ] I understand that the JVM waits for all user threads to finish before exiting naturally.
- [ ] I know that `main()` returning does not, by itself, end the program if other user threads are alive.
- [ ] I understand that `ExecutorService` thread pools default to user (non-daemon) threads.
- [ ] I understand the inheritance rule for daemon status between parent and child threads.

---

## Flashcards

**Q:** What is the default daemon status of a newly created `Thread`? **A:** User (non-daemon) thread — `isDaemon()` returns `false` by default.

**Q:** Does the JVM exit immediately when `main()` returns? **A:** Not necessarily — it waits for all user (non-daemon) threads to finish first.

**Q:** Are `ExecutorService` worker threads user threads or daemon threads by default? **A:** User threads by default — which is why forgetting `shutdown()` can prevent JVM exit.

**Q:** If a user thread creates a child thread without setting daemon status, what is the child's default status? **A:** User thread (inherited from its user-thread parent).

**Q:** Can `main()`'s thread be a daemon thread? **A:** No — it is always a user thread by JVM design.

---

## References

- Oracle Java Tutorials — Thread Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- `java.lang.Thread` API Documentation (`isDaemon`, `setDaemon`)
- `java.util.concurrent.ExecutorService` API Documentation (`shutdown`, `shutdownNow`)