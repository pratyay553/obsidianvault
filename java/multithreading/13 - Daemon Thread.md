# Daemon Thread

## Objective

By the end of this note, you will understand:

- What a daemon thread is and how it differs from a user (non-daemon) thread
- How the JVM decides when to exit based on thread types
- How to create daemon threads and why timing matters
- Where daemon threads are used in real production frameworks

This note pairs directly with [[14 - User Thread]] — the two together explain a subtle but important JVM shutdown behavior.

---

## Why should I learn this?

Many Java developers are surprised to discover that a JVM program can keep running **indefinitely**, even after `main()` has finished, simply because some background thread is still alive. Understanding daemon threads explains exactly when and why this happens — and how to prevent it.

This matters for:

- Writing background/housekeeping threads (like cache cleanup, metrics reporting) that shouldn't prevent your application from shutting down
- Debugging "why won't my JVM process exit?" issues, a surprisingly common real-world annoyance
- Understanding internals of frameworks like the Garbage Collector, which itself runs on daemon threads

---

## Beginner Explanation

Imagine a restaurant at closing time. The manager tells all **staff** (user threads) they must finish their current tasks before they can go home — the restaurant won't lock its doors until every staff member has clocked out.

But there's also a **security guard** (a daemon thread) who patrols the building continuously, with no "finishing point" of their own — they just work as a background presence supporting the restaurant. When the last staff member leaves, the manager doesn't wait around for the security guard to finish some specific task — they simply say "we're closing, guard included" and lock up, and the guard's shift ends automatically along with everything else.

In Java:

- **User threads** (like restaurant staff) — the JVM will wait for all of them to finish before the process exits.
- **Daemon threads** (like the security guard) — the JVM does **not** wait for them; when all user threads finish, the JVM exits immediately, abruptly terminating any still-running daemon threads.

---

## Real World Analogy

**Hospital Analogy**

Think of a hospital that's shutting down for the night:

- **Doctors treating patients** (user threads) — the hospital administration won't lock the doors until every doctor has finished treating their current patients. The hospital's operation is fundamentally tied to completing this essential work.
- **Background HVAC/climate control system** (a daemon thread) — it runs continuously in the background, supporting the hospital's operation, but it's not "essential work" that must be completed before closing. When the last doctor leaves, the building shuts down entirely, including immediately cutting power to the HVAC system — mid-cycle or not.

This distinction — "must finish before shutdown" vs. "just gets cut off when everything else is done" — is exactly the difference between user threads and daemon threads.

---

## Technical Explanation

### Setting a Thread as a Daemon

```java
Thread backgroundThread = new Thread(() -> {
    while (true) {
        System.out.println("Background heartbeat...");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
});

backgroundThread.setDaemon(true); // MUST be called before start()
backgroundThread.start();
```

**Critical rule**: `setDaemon(true)` **must** be called **before** `start()`. Calling it after the thread has already started throws `IllegalThreadStateException`.

### The JVM Shutdown Rule

The JVM exits when **all non-daemon (user) threads have terminated** — regardless of how many daemon threads are still running. When this happens, all remaining daemon threads are **abruptly terminated**, without any guarantee that their code finishes cleanly — no `finally` blocks are guaranteed to run, no cleanup logic is guaranteed to execute.

```java
public class DaemonExitDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread daemon = new Thread(() -> {
            while (true) {
                System.out.println("Daemon still running...");
                try { Thread.sleep(500); } catch (InterruptedException e) { return; }
            }
        });
        daemon.setDaemon(true);
        daemon.start();

        Thread.sleep(2000); // main thread does a bit of work
        System.out.println("Main thread finishing now.");
        // main() returns here -> JVM exits -> daemon thread is abruptly killed
    }
}
```

When you run this, you'll see "Daemon still running..." print a few times, then "Main thread finishing now." — and the program exits immediately, **without** any further daemon output, even though the daemon thread's `while(true)` loop would otherwise run forever.

### Default Inheritance Behavior

A newly created thread **inherits** the daemon status of the thread that created it, **unless** you explicitly override it with `setDaemon()`. So if a daemon thread creates another thread without explicitly setting its daemon status, that new thread is **also** a daemon thread by default (see [[14 - User Thread]] for the full inheritance rule, including `InheritableThreadLocal` considerations in [[61 - InheritableThreadLocal]]).

---

## Internal Working

Step-by-step of how the JVM decides to exit:

1. The JVM tracks all live threads, categorized internally as either **daemon** or **non-daemon (user)**.
2. Every time a thread terminates, the JVM checks: **"are there any non-daemon threads still alive?"**
3. If **yes** → the JVM keeps running, regardless of how many daemon threads remain active.
4. If **no** (all remaining live threads, if any, are daemon threads) → the JVM initiates shutdown.
5. During shutdown, the JVM does **not** wait for daemon threads to finish their current work — it terminates the entire process, and any daemon threads mid-execution are simply cut off.

**Important nuance**: this is different from calling `System.exit()`, which forcibly shuts down the JVM regardless of any threads (daemon or not) — but the "natural" shutdown described above specifically depends on the daemon/non-daemon distinction.

---

## Memory Diagram

```
        JVM Process
   ---------------------------------
   |  User Thread 1  [RUNNING]      |
   |  User Thread 2  [RUNNING]      |
   |  Daemon Thread A [RUNNING]     |
   |  Daemon Thread B [RUNNING]     |
   ---------------------------------

   User Thread 1 finishes... JVM still alive (User Thread 2 remains)
   User Thread 2 finishes... 

        JVM checks: any non-daemon threads left? NO
                        |
                        v
        JVM EXITS IMMEDIATELY
        Daemon Thread A and B are abruptly terminated,
        regardless of what they were doing
```

---

## Execution Flow

```
main() (a user thread) starts
        |
        v
main() creates a daemon thread, sets daemon=true, starts it
        |
        v
main() does its own work, then returns
        |
        v
JVM checks: any non-daemon threads still alive?
        |
       NO
        |
        v
JVM exits immediately
        |
        v
Daemon thread is abruptly terminated mid-execution
```

---

## Code Example

```java
public class DaemonThreadDemo {

    public static void main(String[] args) throws InterruptedException {

        Thread logger = new Thread(() -> {
            int count = 0;
            while (true) {
                System.out.println("Logger tick #" + (++count));
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    System.out.println("Logger interrupted, exiting cleanly.");
                    return;
                }
            }
        }, "Background-Logger");

        System.out.println("Is daemon before setDaemon(): " + logger.isDaemon()); // false by default

        logger.setDaemon(true); // must be called BEFORE start()
        System.out.println("Is daemon after setDaemon(true): " + logger.isDaemon());

        logger.start();

        // Main (user) thread does a small amount of work, then finishes
        for (int i = 1; i <= 3; i++) {
            System.out.println("Main thread doing work: step " + i);
            Thread.sleep(500);
        }

        System.out.println("Main thread finished. JVM should exit now, killing the logger.");
        // No explicit call to stop 'logger' - it just gets cut off when main() returns.
    }
}
```

### Line-by-line explanation

1. `logger.isDaemon()` before `setDaemon()` — prints `false`, since threads are **non-daemon (user threads) by default**, unless created by another daemon thread.
2. `logger.setDaemon(true)` — marks this thread as a daemon; this **must** happen before `.start()`.
3. `logger.start()` — the daemon thread begins its infinite loop, printing a tick roughly every 300ms.
4. The main thread does 3 short steps of "work," each 500ms, then finishes.
5. Because `logger` is a daemon thread, and `main` (the only user thread) is about to finish, **the JVM exits as soon as `main()` returns** — the logger's `while(true)` loop, which would otherwise run forever, gets abruptly cut off mid-execution, with **no** guarantee that "Logger interrupted, exiting cleanly" ever prints.

---

## Dry Run

Approximate output (exact tick count may vary slightly by timing):

```
Is daemon before setDaemon(): false
Is daemon after setDaemon(true): true
Logger tick #1
Main thread doing work: step 1
Logger tick #2
Logger tick #3
Main thread doing work: step 2
Logger tick #4
Logger tick #5
Main thread doing work: step 3
Logger tick #6
Main thread finished. JVM should exit now, killing the logger.
```

Notice: the program ends immediately after the last line — the logger's loop is simply **cut off**, with no further ticks and no "interrupted, exiting cleanly" message, because daemon threads don't get a clean shutdown when the JVM exits this way.

---

## Common Mistakes

1. **Calling `setDaemon(true)` after `start()`.**
    
    - This throws `IllegalThreadStateException` — daemon status must be set before the thread begins running.
2. **Using a daemon thread for critical work that must always complete (e.g., writing important data to disk).**
    
    - Since daemon threads can be abruptly cut off at any point during JVM shutdown, they are **not** appropriate for tasks requiring guaranteed completion or cleanup. Use user (non-daemon) threads, combined with proper shutdown hooks, for such work.
3. **Assuming a daemon thread's `finally` block will always run before shutdown.**
    
    - It will **not** necessarily run — an abrupt daemon thread termination during JVM exit does not guarantee any in-progress cleanup code executes.
4. **Not realizing thread pools (`ExecutorService`) default to non-daemon threads, causing the JVM to hang.**
    
    - A very common real-world "why won't my program exit?" bug: forgetting to call `executorService.shutdown()`. Since the pool's threads are non-daemon by default, the JVM will wait for them indefinitely unless explicitly shut down (see [[46 - ExecutorService]]).

---

## Production Usage

- **The JVM's own Garbage Collector threads** run as daemon threads — they exist to support the application but should never prevent the JVM from exiting once real application work is done.
- **Background metrics/heartbeat reporters** (e.g., periodically sending health metrics to a monitoring system) are commonly implemented as daemon threads, since they're not "core" application logic and shouldn't block shutdown.
- **Scheduled cache-eviction or cleanup threads** in caching libraries are often daemon threads, so they don't prevent the application from shutting down cleanly.
- **Application servers (Tomcat)** carefully manage the daemon status of internal housekeeping threads, ensuring critical request-handling threads are non-daemon (so in-flight requests can complete) while background maintenance threads are daemon.

---

## Interview Questions

**Beginner:**

1. What is a daemon thread?
2. What is the default daemon status of a newly created thread?
3. When must `setDaemon(true)` be called, relative to `start()`?

**Intermediate:** 4. When does the JVM decide to exit, in terms of daemon vs. non-daemon threads? 5. What happens to a daemon thread when the JVM decides to exit? 6. If a daemon thread creates a new thread without setting daemon status explicitly, what is that new thread's default daemon status?

**Advanced:** 7. Why is it risky to perform critical, must-complete work (like writing to a database or file) inside a daemon thread? 8. Why do thread pools created via `ExecutorService` sometimes cause a JVM to "hang" and never exit, and how does this relate to daemon threads?

**Production-Level:** 9. Give a real example of when you'd deliberately use a daemon thread in a production system, and explain why non-daemon would be inappropriate there. 10. How would you diagnose a production Java process that refuses to exit even after all expected application logic has completed?

---

## Follow-up Questions

- Does calling `System.exit()` respect daemon vs. non-daemon thread distinctions? _(No — `System.exit()` forcibly terminates the JVM immediately, regardless of any running threads, daemon or not; it's a much blunter tool than natural JVM shutdown.)_
- Can the main thread itself be made a daemon thread? _(No — the main thread that runs `main()` is always a non-daemon (user) thread by JVM design; you cannot change this.)_
- Are shutdown hooks (`Runtime.addShutdownHook()`) related to daemon threads? _(They're related in purpose — both deal with JVM shutdown behavior — but shutdown hooks are a separate, explicit mechanism for running cleanup code during JVM shutdown, distinct from the daemon/non-daemon thread distinction itself.)_

---

## Best Practices

- Use daemon threads for background, non-critical, "supporting" work (metrics, logging heartbeats, cache cleanup) — never for work that must reliably complete.
- Always call `setDaemon(true)` (if needed) **before** `start()`.
- Be aware that `ExecutorService`-created thread pools default to non-daemon threads — always explicitly `shutdown()` your executors, or you risk your JVM never exiting.
- If you need guaranteed cleanup during shutdown, use `Runtime.getRuntime().addShutdownHook(...)` rather than relying on daemon thread behavior.

---

## Performance Notes

- Daemon vs. non-daemon status has no direct performance impact on the thread's execution itself — it only affects JVM shutdown behavior.
- However, incorrectly leaving non-daemon thread pools running (forgetting to shut them down) can lead to "zombie" JVM processes that consume system resources indefinitely in production, which is an operational/performance concern at the infrastructure level.

---

## Comparison Table

|Aspect|User Thread (non-daemon)|Daemon Thread|
|---|---|---|
|JVM waits for it before exiting?|Yes|No|
|Default status for new threads|Yes (non-daemon by default)|No (must opt in via `setDaemon(true)`)|
|Guaranteed clean shutdown/cleanup?|Yes (JVM waits for it to finish)|No (can be abruptly cut off)|
|Typical use case|Core application logic|Background/support tasks (GC, metrics, cleanup)|
|Can be set after `.start()`?|N/A|No — must be set before `.start()`|

---

## Summary

- A **daemon thread** is a background thread that does not prevent the JVM from exiting.
- The JVM exits once all **non-daemon (user)** threads have finished, regardless of how many daemon threads remain alive — and any remaining daemon threads are then abruptly terminated.
- `setDaemon(true)` must be called **before** `.start()`, or an exception is thrown.
- Daemon threads are appropriate for background/support work (metrics, cleanup, GC) but never for critical work requiring guaranteed completion.
- Threads created by a daemon thread inherit daemon status by default, unless explicitly overridden.

---

## Cheat Sheet

|Rule|Detail|
|---|---|
|Set daemon status|`thread.setDaemon(true)`, must be BEFORE `.start()`|
|Default status|Non-daemon (unless created by a daemon thread)|
|JVM exit condition|All non-daemon threads have finished|
|Daemon thread on exit|Abruptly terminated, no guaranteed cleanup|
|Common real-world bug|Forgetting to shut down an `ExecutorService` (non-daemon by default)|

---

## Keywords

Daemon Thread, User Thread, JVM Shutdown, setDaemon(), IllegalThreadStateException, Garbage Collector Threads, ExecutorService Shutdown

---

## Related Notes

[[14 - User Thread]] [[09 - Thread Lifecycle]] [[46 - ExecutorService]] [[67 - Garbage Collection and Threads]]

---

## Practice Exercises

**Easy:**

1. Create a thread with an infinite loop, mark it as a daemon, and observe that the program exits cleanly once `main()` finishes, without waiting for the loop.
2. Try calling `setDaemon(true)` _after_ `.start()` and observe the exception thrown.

**Medium:** 3. Create an `ExecutorService` (which defaults to non-daemon threads), submit a long-running task, and observe that your program does **not** exit even after `main()` finishes — until you either call `shutdown()` or the task completes. 4. Create a daemon thread that creates another thread inside it (without explicitly setting daemon status on the child) — confirm the child thread is also a daemon by default.

**Hard:** 5. Build a small program with both a critical non-daemon "must complete" task and a daemon "background heartbeat" task, and demonstrate that the JVM correctly waits for the critical task but not the heartbeat.

---

## Mini Project

**Project: Background Heartbeat Service**

Build a small console application that:

1. Simulates "processing 5 orders" on the main (user) thread, each taking 1 second.
2. Runs a daemon thread in the background that prints a heartbeat message every 500ms, simulating a health-check reporter.
3. Confirms that once all 5 orders are "processed" and `main()` finishes, the program exits immediately — the heartbeat daemon thread does not keep it running.
4. As a bonus, add a **second**, non-daemon thread doing a genuinely important cleanup task, and confirm the JVM correctly waits for it before exiting, unlike the daemon heartbeat.

**Goal:** Build a clear, hands-on mental model of exactly when the JVM decides to exit, and why daemon threads behave differently from user threads in that decision.

---

## Common Interview Trap

**Trap Question:** _"If I create a daemon thread that writes important data to a file every few seconds, is it guaranteed to save its last piece of data before the JVM exits?"_

The trap is assuming daemon threads get any kind of "grace period" or clean shutdown when the JVM exits. The correct answer: **no** — daemon threads can be abruptly terminated at any point, including mid-write, when the JVM decides to exit (because all non-daemon threads have finished). There's no guarantee that in-progress work, `finally` blocks, or any cleanup logic inside a daemon thread will complete. If data must reliably be saved, that work belongs on a **non-daemon** thread, or should be coordinated through an explicit `Runtime.addShutdownHook()` with appropriate care.

---

## Frequently Asked Questions

**Q: Are daemon threads slower or less capable than user threads?** A: No — daemon and non-daemon threads run identically in terms of execution capability and performance; the only difference is how they affect JVM shutdown behavior.

**Q: Can I convert a running non-daemon thread into a daemon thread mid-execution?** A: No — `setDaemon()` can only be called before the thread starts; once started, its daemon status is fixed for its lifetime.

**Q: Does `Thread.sleep()` or being `BLOCKED`/`WAITING` prevent a thread from being counted for JVM shutdown purposes?** A: No — a thread's daemon/non-daemon status matters for shutdown decisions regardless of its current lifecycle state; a non-daemon thread that's `WAITING` indefinitely will still keep the JVM alive indefinitely.

**Q: Is the Garbage Collector thread a daemon thread?** A: Yes — GC threads are daemon threads, since they exist to support the application but should never be the reason the JVM fails to exit.

---

## Revision Checklist

- [ ] I can explain what a daemon thread is and how it differs from a user thread.
- [ ] I understand exactly when the JVM decides to exit.
- [ ] I know that `setDaemon(true)` must be called before `.start()`.
- [ ] I understand that daemon threads can be abruptly terminated with no cleanup guarantee.
- [ ] I know that `ExecutorService` thread pools default to non-daemon threads, and why this matters practically.

---

## Flashcards

**Q:** What is a daemon thread? **A:** A background thread that does not prevent the JVM from exiting once all non-daemon threads finish.

**Q:** When must `setDaemon(true)` be called? **A:** Before `.start()` — otherwise it throws `IllegalThreadStateException`.

**Q:** What happens to daemon threads when the JVM exits? **A:** They are abruptly terminated, with no guarantee that in-progress work or cleanup code completes.

**Q:** What is the default daemon status of a newly created thread? **A:** Non-daemon, unless it was created by a daemon thread (in which case it inherits daemon status by default).

**Q:** Are threads created by `ExecutorService` daemon or non-daemon by default? **A:** Non-daemon — which is why forgetting to call `shutdown()` can prevent a JVM from exiting.

---

## References

- Oracle Java Tutorials — Thread Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- `java.lang.Thread` API Documentation (`setDaemon`, `isDaemon`)
- `java.lang.Runtime` API Documentation (`addShutdownHook`)