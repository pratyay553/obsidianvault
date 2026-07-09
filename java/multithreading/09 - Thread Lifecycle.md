# Thread Lifecycle

## Objective

By the end of this note, you will understand:

- Every state a Java thread can be in, from birth to death
- What triggers transitions between these states
- How the JVM and OS cooperate to manage this lifecycle
- How to reason about a thread's current state when debugging production issues

This note opens **Section 3: Thread Lifecycle & Control** and is the conceptual foundation for [[10 - Thread States]], [[11 - Thread Scheduler]], and [[12 - Context Switching]].

---

## Why should I learn this?

Every concurrency bug you'll ever debug — a deadlock, a thread stuck forever, a thread that never starts — ultimately comes down to understanding **what state a thread is in, and why it's stuck there**. Thread dumps (see [[69 - Thread Dump Analysis]]), one of the most important production debugging tools for Java engineers, are essentially just **snapshots of every thread's current lifecycle state**.

If you don't deeply understand the lifecycle, thread dumps will look like meaningless walls of text. If you do understand it, they become a powerful diagnostic tool that can pinpoint the exact line of code where a thread is stuck.

---

## Beginner Explanation

Just like a human goes through life stages — born, growing, working, resting, eventually passing away — a Java thread goes through a defined set of **states** during its lifetime:

1. **New** — the thread object has been created, but `.start()` hasn't been called yet.
2. **Runnable** — the thread is ready to run (or currently running) — it's competing for CPU time with other threads.
3. **Blocked** — the thread is waiting to acquire a lock held by another thread.
4. **Waiting** — the thread is waiting indefinitely for another thread to do something (e.g., call `notify()`).
5. **Timed Waiting** — like Waiting, but with a maximum time limit (e.g., `Thread.sleep(1000)`).
6. **Terminated** — the thread has finished executing (or was stopped due to an exception).

Every thread you ever create moves through some subset of these states, in a specific, well-defined order — it can never skip backward illegally (e.g., a Terminated thread can never come back to New).

---

## Real World Analogy

**Traffic Signal / Airport Boarding Analogy**

Think of a thread's lifecycle like a passenger's journey through an airport:

- **New** = You've booked a ticket but haven't arrived at the airport yet.
- **Runnable** = You've checked in and are in line, ready to board (you're competing with other passengers for the gate agent's attention — just like threads compete for CPU time).
- **Running** (a sub-state of Runnable) = You're actively boarding the plane right now.
- **Blocked** = You're stuck at security because someone ahead of you is taking forever (you need a resource — the security lane — that's currently occupied).
- **Waiting** = You're waiting in the lounge for a friend to text you "let's board" — with no defined time limit; you'll wait indefinitely until that signal comes.
- **Timed Waiting** = You've set an alarm to wake up in exactly 20 minutes, regardless of whether anything else happens.
- **Terminated** = You've landed and left the airport — your journey (the thread's execution) is over.

---

## Technical Explanation

Java exposes these states through the enum `Thread.State`:

```java
public enum State {
    NEW,
    RUNNABLE,
    BLOCKED,
    WAITING,
    TIMED_WAITING,
    TERMINATED
}
```

You can check any thread's current state at any time using:

```java
Thread.State state = someThread.getState();
```

Let's go through each state precisely.

### 1. NEW

- The `Thread` object has been created (`new Thread(...)`), but `.start()` has **not** yet been called.
- No OS-level thread resources have been allocated yet.

### 2. RUNNABLE

- After `.start()` is called, the thread enters `RUNNABLE`.
- **Important nuance**: `RUNNABLE` in Java covers **both** "ready to run, waiting for CPU time" and "actually currently running on a CPU core" — Java's `Thread.State` enum does not distinguish between these two OS-level sub-states. The actual decision of which runnable thread gets CPU time next belongs to the OS scheduler (see [[11 - Thread Scheduler]]).

### 3. BLOCKED

- The thread is trying to enter a `synchronized` block/method, but another thread currently holds the lock (monitor) it needs.
- The thread will remain `BLOCKED` until the lock becomes available (see [[23 - Monitor Lock]], [[25 - synchronized Keyword]]).

### 4. WAITING

- The thread called one of these methods, with **no timeout**:
    - `Object.wait()` (no arguments)
    - `Thread.join()` (no arguments)
    - `LockSupport.park()`
- It will remain in this state **indefinitely** until another thread performs a specific action (like calling `notify()`, or the joined thread finishing).

### 5. TIMED_WAITING

- Same as `WAITING`, but with a **specified maximum time**. The thread will automatically wake up and resume even if no external signal ever comes. Triggered by:
    - `Thread.sleep(millis)`
    - `Object.wait(timeout)`
    - `Thread.join(timeout)`
    - `LockSupport.parkNanos(...)` / `parkUntil(...)`

### 6. TERMINATED

- The thread has completed execution, either because its `run()` method returned normally, or because an uncaught exception propagated out of it.
- A `TERMINATED` thread cannot be restarted — calling `.start()` again throws `IllegalThreadStateException`.

---

## Internal Working

Here's what actually drives each state transition, step by step:

1. **NEW → RUNNABLE**: Triggered by calling `.start()`. The JVM requests a new OS-level thread; once the OS accepts the request and the thread is eligible for scheduling, its Java-visible state becomes `RUNNABLE`.
2. **RUNNABLE → BLOCKED**: Happens when the thread tries to enter a `synchronized` block/method whose monitor lock is already held by another thread.
3. **BLOCKED → RUNNABLE**: Happens when the lock becomes available and this thread acquires it (the JVM/OS decides which waiting thread gets the lock next, though this is not strictly guaranteed to be fair/ordered — see [[38 - Starvation]]).
4. **RUNNABLE → WAITING**: Happens when the thread calls `wait()`, `join()` (no timeout), or `LockSupport.park()`.
5. **WAITING → RUNNABLE**: Happens when another thread calls `notify()`/`notifyAll()` (for `wait()`), the joined thread finishes (for `join()`), or `LockSupport.unpark()` is called.
6. **RUNNABLE → TIMED_WAITING**: Happens via `sleep(ms)`, `wait(ms)`, `join(ms)`, or the timed `LockSupport` variants.
7. **TIMED_WAITING → RUNNABLE**: Happens either when the timeout expires, OR when the relevant "wake up" signal arrives early (e.g., `notify()` before the timeout for `wait(ms)`).
8. **RUNNABLE → TERMINATED**: Happens when `run()` completes (normally or via an uncaught exception).

---

## Memory Diagram

```
             new Thread(...)
                   |
                   v
     +-----------------------------+
     |            NEW               |
     +-----------------------------+
                   |  .start()
                   v
     +-----------------------------+
     |          RUNNABLE            |<----------------+
     +-----------------------------+                  |
        |         |         |                          |
  needs a lock  wait()   sleep()/wait(ms)/join(ms)     |
   (busy)         |            |                        |
        v         v            v                        |
  +----------+ +---------+ +----------------+            |
  | BLOCKED  | | WAITING | | TIMED_WAITING  |------------+
  +----------+ +---------+ +----------------+  (timeout or signal)
        |             |
  (lock acquired) (notify/join done)
        |             |
        +------> RUNNABLE <-------+

     run() finishes (normally or via exception)
                   |
                   v
     +-----------------------------+
     |         TERMINATED           |
     +-----------------------------+
```

---

## Execution Flow

Let's trace a concrete thread through multiple states in one lifetime:

```
1. Thread t = new Thread(task);        --> state: NEW
2. t.start();                          --> state: RUNNABLE
3. Inside task, thread calls
   synchronized block held by another
   thread                              --> state: BLOCKED
4. Lock becomes free, thread acquires
   it and continues                    --> state: RUNNABLE
5. Thread calls Thread.sleep(2000)     --> state: TIMED_WAITING
6. 2 seconds pass                      --> state: RUNNABLE
7. Thread calls someObject.wait()      --> state: WAITING
8. Another thread calls
   someObject.notify()                 --> state: RUNNABLE
9. task's run() method returns         --> state: TERMINATED
```

---

## Code Example

```java
public class ThreadLifecycleDemo {

    static final Object lock = new Object();

    public static void main(String[] args) throws InterruptedException {

        Thread worker = new Thread(() -> {
            synchronized (lock) {
                try {
                    System.out.println("Worker entering TIMED_WAITING via sleep...");
                    Thread.sleep(1000);

                    System.out.println("Worker entering WAITING via wait()...");
                    lock.wait(); // releases the lock while waiting
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            System.out.println("Worker finishing...");
        });

        System.out.println("State right after creation: " + worker.getState()); // NEW

        worker.start();
        Thread.sleep(200); // give worker time to start and enter sleep
        System.out.println("State during sleep: " + worker.getState()); // TIMED_WAITING

        Thread.sleep(1000); // wait past the sleep, worker should now be in WAITING
        System.out.println("State during wait(): " + worker.getState()); // WAITING

        synchronized (lock) {
            lock.notify(); // wake up the worker
        }

        worker.join(); // wait for worker to fully terminate
        System.out.println("State after termination: " + worker.getState()); // TERMINATED
    }
}
```

### Line-by-line explanation

1. `worker.getState()` right after creation, before `.start()`, prints `NEW`.
2. After `.start()` and a short pause, the worker is inside `Thread.sleep(1000)`, so its state is `TIMED_WAITING`.
3. After the sleep completes, the worker calls `lock.wait()` (releasing the lock while waiting — see [[40 - wait()]]), so its state becomes `WAITING`.
4. The main thread acquires the same lock and calls `lock.notify()`, waking the worker up, transitioning it back to `RUNNABLE` (specifically, it first needs to re-acquire the lock, so technically it may briefly be `BLOCKED` if the main thread hasn't released the lock yet — a subtlety worth noting).
5. After `run()` finishes, `worker.getState()` prints `TERMINATED`.

---

## Dry Run

|Time|Main Thread Action|Worker Thread State|
|---|---|---|
|0ms|Creates `worker`, checks state|`NEW`|
|0ms|Calls `worker.start()`|`RUNNABLE` (briefly, then enters sync block)|
|200ms|Checks state (worker is sleeping)|`TIMED_WAITING`|
|1200ms|Checks state (worker is now waiting)|`WAITING`|
|1200ms+|Calls `lock.notify()`|Worker transitions toward `RUNNABLE`|
|shortly after|Worker finishes `run()`|`TERMINATED`|

---

## Common Mistakes

1. **Confusing `BLOCKED` with `WAITING`.**
    
    - `BLOCKED` specifically means "waiting to acquire a lock to enter a `synchronized` block." `WAITING` means "voluntarily paused, waiting for an explicit signal (like `notify()`) or another thread's completion (`join()`)." These are different states with different causes, and interviewers frequently test this distinction.
2. **Assuming `RUNNABLE` means "currently executing on a CPU."**
    
    - `RUNNABLE` just means "eligible to run" — the thread might be sitting in the OS scheduler's queue, waiting its turn for actual CPU time. Java's `Thread.State` enum does not expose a separate "currently executing" state.
3. **Thinking a `TERMINATED` thread can be restarted.**
    
    - Once terminated, a `Thread` object is permanently done. You must create a brand-new `Thread` object (often wrapping the same `Runnable`/`Callable`) to run the logic again.
4. **Forgetting that `wait()` releases the lock, but `sleep()` does not.**
    
    - This is a critical, frequently-tested distinction: `Thread.sleep()` holds onto any locks it currently owns while sleeping, whereas `Object.wait()` **releases** the lock on the object it's called on, allowing other threads to acquire it during the wait (see [[40 - wait()]], [[18 - Sleep()]]).

---

## Production Usage

- **Thread dump analysis** (see [[69 - Thread Dump Analysis]]) relies entirely on understanding these states — a production incident investigation often starts with "what state are all our threads in right now, and why are so many `BLOCKED`?"
- **Deadlock detection tools** (built into tools like VisualVM, JConsole, or `jstack`) work by analyzing which threads are `BLOCKED` waiting on locks held by other `BLOCKED` threads, forming a cycle.
- **Connection pool exhaustion diagnosis**: If many application threads show as `WAITING` or `TIMED_WAITING` on a connection pool's internal lock, this often indicates the pool is too small for the current load.
- **Health checks and monitoring**: Some systems periodically sample thread states to detect unusually high numbers of `BLOCKED` threads, which can be an early warning sign of lock contention problems before they become full outages.

---

## Interview Questions

**Beginner:**

1. What are the six states in Java's `Thread.State` enum?
2. What state is a thread in immediately after being created but before `.start()` is called?
3. What state is a thread in after `run()` finishes?

**Intermediate:** 4. What is the difference between `BLOCKED` and `WAITING`? 5. What is the difference between `WAITING` and `TIMED_WAITING`? 6. Does `Thread.sleep()` release any locks the thread currently holds?

**Advanced:** 7. Can a thread go directly from `NEW` to `TERMINATED` without ever being `RUNNABLE`? (Answer: extremely unlikely in practice, but if `run()` executes near-instantly, the `RUNNABLE` state may simply not be observed/sampled — it still technically passes through it.) 8. Why doesn't Java's `Thread.State` enum distinguish between "ready to run" and "actually running on a CPU core"?

**Production-Level:** 9. If a thread dump shows 50 threads `BLOCKED` on the same lock, what does that tell you about your system's design? 10. How would you use thread state information to diagnose whether a production slowdown is caused by lock contention versus simply too much CPU-bound work?

---

## Follow-up Questions

- If Thread A calls `join()` on Thread B, what state does Thread A enter? _(`WAITING`, if no timeout is specified, or `TIMED_WAITING` if `join(millis)` is used — Thread A stays in that state until Thread B terminates or the timeout expires.)_
- Can a `BLOCKED` thread be interrupted out of its blocked state? _(No — a thread `BLOCKED` waiting to acquire a `synchronized` lock cannot be interrupted out of that wait; this is a key limitation that `ReentrantLock`'s `lockInterruptibly()` was designed to address — see [[32 - ReentrantLock]].)_
- Does calling `Thread.yield()` change a thread's `Thread.State`? _(No — `yield()` is a hint to the scheduler, but the thread remains in `RUNNABLE` throughout; see [[19 - Yield()]].)_

---

## Best Practices

- When debugging concurrency issues, always start by taking a thread dump and categorizing threads by state — this quickly narrows down whether you're facing a lock contention issue (`BLOCKED`), a hung dependency (`WAITING`/`TIMED_WAITING`), or genuine CPU saturation (many `RUNNABLE` threads, high CPU usage).
- Avoid designs that rely on many threads competing for a single shared lock, as this leads to large numbers of `BLOCKED` threads and poor throughput.
- Prefer `ReentrantLock` with timeouts (`tryLock(timeout, unit)`) over indefinite `synchronized` blocking, in scenarios where you need to detect and recover from lock contention gracefully (see [[32 - ReentrantLock]]).

---

## Performance Notes

- Threads sitting `BLOCKED` or `WAITING` are not consuming CPU cycles while in that state (the OS suspends them), but they still hold onto memory resources (their stack) and can indicate architectural bottlenecks even without direct CPU cost.
- Excessive transitions between `RUNNABLE` and `BLOCKED`/`WAITING` (thrashing) can indicate poor lock granularity, leading to overhead beyond just the "waiting" time itself — see [[72 - Performance Tuning]].

---

## Comparison Table

|State|Triggered By|Releases Held Locks?|Wakes Up Automatically?|
|---|---|---|---|
|NEW|Thread created, not started|N/A|N/A|
|RUNNABLE|`.start()` called|N/A|N/A|
|BLOCKED|Waiting to enter `synchronized` block|N/A (it doesn't hold the lock yet)|Yes, once lock is free|
|WAITING|`wait()`, `join()`, `park()` (no timeout)|Yes (for `wait()`)|No — needs explicit signal|
|TIMED_WAITING|`sleep()`, `wait(ms)`, `join(ms)`|Yes for `wait(ms)`; No for `sleep()`|Yes, after timeout (or earlier signal)|
|TERMINATED|`run()` completes|N/A|N/A|

---

## Summary

- Every Java thread moves through a well-defined lifecycle: `NEW → RUNNABLE → (BLOCKED/WAITING/TIMED_WAITING) → RUNNABLE → TERMINATED`.
- `BLOCKED` specifically means waiting for a lock; `WAITING`/`TIMED_WAITING` mean voluntarily paused for a signal or timeout.
- `Thread.sleep()` does **not** release held locks; `Object.wait()` **does**.
- Understanding these states is essential for reading thread dumps and diagnosing real production concurrency issues.
- A `TERMINATED` thread cannot be restarted — a new `Thread` object is required to run the same logic again.

---

## Cheat Sheet

|Trigger|Resulting State|
|---|---|
|`new Thread(...)`|NEW|
|`.start()`|RUNNABLE|
|Waiting for a `synchronized` lock|BLOCKED|
|`wait()`, `join()`, `park()` (no timeout)|WAITING|
|`sleep(ms)`, `wait(ms)`, `join(ms)`|TIMED_WAITING|
|`run()` completes|TERMINATED|

---

## Keywords

Thread.State, NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED, Lifecycle, Thread Dump

---

## Related Notes

[[10 - Thread States]] [[11 - Thread Scheduler]] [[18 - Sleep()]] [[20 - Join()]] [[40 - wait()]] [[69 - Thread Dump Analysis]]

---

## Practice Exercises

**Easy:**

1. Create a thread, print its state before `.start()`, immediately after `.start()`, and after it finishes.
2. Create a thread that sleeps for 2 seconds, and print its state from the main thread while it's sleeping.

**Medium:** 3. Create two threads that both try to enter the same `synchronized` block, where one holds the lock for 3 seconds. Print the second thread's state while it waits — confirm it shows `BLOCKED`. 4. Create a thread that calls `wait()` on a shared object, then have the main thread print its state before calling `notify()`.

**Hard:** 5. Write a program with 2 threads that deadlock each other (each holding a lock the other needs — full detail in [[36 - Deadlock]]), and print both threads' states, confirming they both show `BLOCKED` indefinitely.

---

## Mini Project

**Project: Thread State Monitor**

Build a small console application that:

1. Creates a worker thread that goes through several states deliberately: sleeps (`TIMED_WAITING`), waits on a lock held by another busy thread (`BLOCKED`), then calls `wait()` (`WAITING`), and finally completes (`TERMINATED`).
2. From the main thread, runs a loop that samples and prints the worker's `getState()` every 100ms.
3. Produces a full timeline log showing every state the thread passed through and roughly how long it spent in each.

**Goal:** Build direct, hands-on familiarity with reading and interpreting thread states — the exact skill needed for real thread dump analysis.

---

## Common Interview Trap

**Trap Question:** _"If a thread is in the `RUNNABLE` state, does that mean it's currently executing on a CPU core right now?"_

The trap is assuming `RUNNABLE` means "actively running." In reality, Java's `Thread.State.RUNNABLE` covers **both** "waiting in the OS scheduler's queue for CPU time" and "actually executing on a core right now" — Java does not expose a separate state for these two situations. The actual decision of which `RUNNABLE` thread gets CPU time is made entirely by the **OS scheduler**, not the JVM (see [[11 - Thread Scheduler]]), and Java's `Thread.State` API has no visibility into that OS-level distinction.

---

## Frequently Asked Questions

**Q: Can a thread move from `BLOCKED` directly to `TERMINATED` without passing through `RUNNABLE` again?** A: No — once a `BLOCKED` thread acquires the lock it was waiting for, it transitions back to `RUNNABLE` first (even if only briefly) before it can proceed to complete and reach `TERMINATED`.

**Q: Is there a "PAUSED" or "SUSPENDED" state in modern Java?** A: No — older, now-deprecated methods like `Thread.suspend()` and `Thread.resume()` existed in early Java versions but are deprecated due to deadlock-prone behavior; they are not part of the standard `Thread.State` lifecycle model.

**Q: Does calling `Thread.interrupt()` change a thread's state directly?** A: It depends on the current state — interrupting a thread in `WAITING`/`TIMED_WAITING` (on an interruptible wait) typically causes it to wake up and throw `InterruptedException`, transitioning it back toward `RUNNABLE`. Interrupting a `RUNNABLE` thread just sets an internal "interrupted" flag, which the thread's own code must choose to check (see [[21 - Interrupt()]]).

**Q: How can I check a thread's state from within a running production application?** A: Via `Thread.getState()` programmatically, or externally via tools like `jstack`, VisualVM, or JConsole, which produce full thread dumps showing every thread's current state (see [[69 - Thread Dump Analysis]]).

---

## Revision Checklist

- [ ] I can list all six `Thread.State` values in the correct conceptual order.
- [ ] I can explain the difference between `BLOCKED` and `WAITING`.
- [ ] I understand that `sleep()` does not release locks, but `wait()` does.
- [ ] I understand that `RUNNABLE` does not guarantee the thread is currently executing on a CPU.
- [ ] I can trace a thread's full lifecycle through a piece of sample code.

---

## Flashcards

**Q:** What state is a `Thread` object in immediately after `new Thread(...)`, before `.start()`? **A:** `NEW`

**Q:** What state does a thread enter when trying to acquire a `synchronized` lock held by another thread? **A:** `BLOCKED`

**Q:** What's the key difference between `WAITING` and `TIMED_WAITING`? **A:** `WAITING` has no time limit and requires an explicit signal to resume; `TIMED_WAITING` automatically resumes after a specified timeout.

**Q:** Does `Thread.sleep()` release any locks the thread holds? **A:** No.

**Q:** Does `Object.wait()` release the lock on the object it's called on? **A:** Yes.

---

## References

- Oracle Java Tutorials — Thread States: https://docs.oracle.com/javase/8/docs/api/java/lang/Thread.State.html
- `java.lang.Thread` API Documentation
- `java.lang.Thread.State` Enum Documentation