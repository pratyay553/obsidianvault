# Interrupt()

## Objective

By the end of this note, you will understand:

- What thread interruption actually is (and what it is NOT)
- The critical distinction between "interrupting" and "forcibly stopping" a thread
- How the interrupted status flag works, and how different blocking methods respond to it
- How to write interruption-aware code that cooperates properly with cancellation

This note closes out **Section 3: Thread Lifecycle & Control**, tying together everything from [[09 - Thread Lifecycle]] through [[20 - Join()]].

---

## Why should I learn this?

Interruption is Java's **only built-in, standard mechanism for requesting that a thread stop what it's doing**. Yet it's one of the most widely misunderstood mechanisms in the entire language — many developers assume `interrupt()` forcibly kills a thread, when in reality it does something much gentler and more cooperative. Getting this wrong leads to two common, opposite failure modes: threads that never actually stop when asked, or threads that lose important state because interruption was handled carelessly.

Understanding interruption correctly is essential for graceful shutdown logic, cancellable tasks, and correctly implementing `Future.cancel()` behavior (see [[07 - Future]]).

---

## Beginner Explanation

Calling `thread.interrupt()` does **not** forcibly stop the thread — despite what the name might suggest. It's more like **tapping someone on the shoulder** to get their attention: "Hey, when you get a chance, you might want to stop what you're doing." What that person does next is entirely up to **them** — they might immediately stop, finish their current sentence first, or (if they're not paying attention) not notice the tap at all and keep going.

In Java terms: calling `interrupt()` simply **sets an internal flag** on the target thread (its "interrupted status"), and — if the thread happens to be blocked in certain specific methods (like `sleep()`, `wait()`, or `join()`) — wakes it up early with an `InterruptedException`. But if the thread's code never checks this flag or isn't currently in one of those specific blocking calls, **nothing happens** — the thread just keeps running, completely unaware it was "interrupted."

---

## Real World Analogy

**Courier Service Analogy**

Imagine dispatch radios a courier mid-delivery: "Hey, if it's convenient, please head back to base when you get a chance." The courier:

- If they're currently stopped at a red light (analogous to being in a blocking call like `sleep()`), they might immediately hear the message and decide to turn around right then.
- If they're in the middle of handing a package to a customer (a critical, uninterruptible action, like a tight computational loop), they'll naturally finish that handoff first before checking their radio again.
- If their radio happens to be turned down too low or broken (analogous to code that never checks `isInterrupted()`), they might never even hear the message and just keep working, completely oblivious.

Dispatch cannot force the courier to abandon their van remotely — they can only send the request and hope the courier is designed to listen and respond appropriately.

---

## Technical Explanation

### The Interrupt Mechanism: Just a Flag

Every `Thread` object has an internal boolean **interrupted status flag**, initially `false`. Calling `.interrupt()` on a thread does two things:

1. Sets that thread's interrupted status flag to `true`.
2. **If** the thread is currently blocked inside certain specific "interruptible" methods (`Thread.sleep()`, `Object.wait()`, `Thread.join()`, and several others in `java.util.concurrent` like `BlockingQueue.take()`), that method **immediately throws `InterruptedException`**, and — importantly — **the interrupted status flag is cleared back to `false`** as part of this process.

```java
Thread worker = new Thread(() -> {
    // does something
});
worker.start();
worker.interrupt(); // sets the flag; may also wake it up if blocked appropriately
```

### Checking Interrupted Status

```java
// Instance method - checks a specific thread, does NOT clear the flag
boolean interrupted = someThread.isInterrupted();

// Static method - checks the CURRENT thread, and CLEARS the flag afterward
boolean wasInterrupted = Thread.interrupted();
```

This distinction (`isInterrupted()` vs. the static `Thread.interrupted()`) is a classic point of confusion:

- `someThread.isInterrupted()` — an **instance** method, simply reads the flag without modifying it.
- `Thread.interrupted()` — a **static** method, always checks the **currently executing thread**, and **clears** the flag as a side effect of checking it.

### Correctly Writing Interruptible Code

For code with a loop doing ongoing work, the standard pattern is:

```java
public void run() {
    while (!Thread.currentThread().isInterrupted()) {
        // do a unit of work
    }
    System.out.println("Detected interruption, stopping gracefully.");
}
```

For code that calls a blocking method like `sleep()` inside the loop:

```java
public void run() {
    try {
        while (!Thread.currentThread().isInterrupted()) {
            // do a unit of work
            Thread.sleep(100); // this itself throws InterruptedException if interrupted
        }
    } catch (InterruptedException e) {
        System.out.println("Interrupted during sleep, stopping gracefully.");
        Thread.currentThread().interrupt(); // best practice: re-set the flag (see below)
    }
}
```

### Why Re-Interrupting Is Considered Best Practice

When `InterruptedException` is caught, the interrupted flag has already been **cleared** by the JVM as part of throwing that exception. If your `catch` block doesn't propagate this information somehow, and this code is itself called from within a larger method, any code **further up the call stack** has no way of knowing an interruption occurred at all. Calling `Thread.currentThread().interrupt()` inside the catch block **re-sets** the flag, preserving the signal for any calling code that might also want to check for and respond to interruption.

### What Interruption is NOT

- It is **not** a forceful stop — there is no way to make a thread stop against its will using standard interruption (the old `Thread.stop()` method that did this forcibly is deprecated precisely because it's dangerous — see [[17 - Thread Groups]]).
- It does **not** automatically release any locks the thread holds.
- It has **no effect** on a thread's code that never checks `isInterrupted()` and never calls an interruptible blocking method.

---

## Internal Working

Step-by-step of what happens when `.interrupt()` is called:

1. Some thread (call it Thread A) calls `targetThread.interrupt()`.
2. The JVM sets `targetThread`'s internal interrupted flag to `true`.
3. The JVM checks: is `targetThread` currently blocked inside an interruptible method (`sleep()`, `wait()`, `join()`, certain `java.util.concurrent` blocking calls)?
4. **If yes**: that blocking call immediately throws `InterruptedException` inside `targetThread`, and the flag is cleared back to `false` as part of this.
5. **If no** (the thread is doing normal, non-blocking work, like a tight loop): nothing happens immediately — the flag is simply set to `true` and sits there until `targetThread`'s own code happens to check `isInterrupted()` (or later calls one of those interruptible blocking methods, which will then immediately see the already-set flag and throw `InterruptedException` right away).
6. It is entirely up to `targetThread`'s own code to decide what to do once it notices the flag — stop gracefully, log something, clean up resources, etc. Nothing is forced.

---

## Memory Diagram

```
        Thread object (targetThread)
   -----------------------------------
   |  interrupted flag: false -> true |  <- set by interrupt() call
   -----------------------------------
                |
                v
   Is targetThread currently blocked in
   sleep()/wait()/join()/etc.?
        |                    |
       YES                   NO
        |                    |
        v                    v
  InterruptedException   Flag just sits as 'true'
  thrown immediately,    until the thread's own
  flag CLEARED back      code checks isInterrupted()
  to false               or hits a blocking call later
```

---

## Execution Flow

```
Thread A calls targetThread.interrupt()
        |
        v
targetThread's interrupted flag set to true
        |
   -----------------------------
   |                           |
   v                           v
targetThread is blocked    targetThread is doing
in sleep/wait/join         normal (non-blocking) work
   |                           |
   v                           v
InterruptedException       Flag remains true until
thrown immediately          targetThread's own code
(flag cleared)               checks isInterrupted()
   |                           |
   v                           v
targetThread's catch       targetThread's loop condition
block decides what to do   (or later blocking call) detects
                            it and decides what to do
```

---

## Code Example

```java
public class InterruptDemo {

    public static void main(String[] args) throws InterruptedException {

        // Scenario 1: interrupting a thread doing CPU work in a loop (checks isInterrupted())
        Thread cpuWorker = new Thread(() -> {
            long count = 0;
            while (!Thread.currentThread().isInterrupted()) {
                count++;
            }
            System.out.println("CPU worker stopped gracefully. Counted to: " + count);
        }, "CPU-Worker");

        cpuWorker.start();
        Thread.sleep(50); // let it run a bit
        cpuWorker.interrupt(); // flag is set; loop condition will detect it soon
        cpuWorker.join();

        // Scenario 2: interrupting a thread blocked in sleep()
        Thread sleepyWorker = new Thread(() -> {
            try {
                System.out.println("Sleepy worker going to sleep for 10 seconds...");
                Thread.sleep(10000);
                System.out.println("Sleepy worker woke up naturally (should NOT happen here).");
            } catch (InterruptedException e) {
                System.out.println("Sleepy worker was interrupted during sleep, stopping gracefully.");
                Thread.currentThread().interrupt(); // re-set flag, good practice
            }
        }, "Sleepy-Worker");

        sleepyWorker.start();
        Thread.sleep(500); // let it start sleeping
        sleepyWorker.interrupt(); // wakes it up immediately via InterruptedException
        sleepyWorker.join();

        System.out.println("Demo complete.");
    }
}
```

### Line-by-line explanation

1. `cpuWorker` runs a tight loop, checking `Thread.currentThread().isInterrupted()` on every iteration — this is the correct pattern for **non-blocking** interruptible work.
2. After 50ms, `cpuWorker.interrupt()` is called — the flag is set, and on the **next** loop iteration, the `while` condition detects it and the loop exits gracefully.
3. `sleepyWorker` demonstrates the **blocking** case: it's inside `Thread.sleep(10000)` when interrupted, so it doesn't have to wait for a loop check — the `sleep()` call itself immediately throws `InterruptedException`.
4. Both scenarios show the thread's own code being responsible for deciding how to respond — `interrupt()` itself never forces anything.

---

## Dry Run

**Scenario 1 (CPU worker):**

|Time|Event|
|---|---|
|0ms|`cpuWorker` starts, looping and counting|
|50ms|`main` calls `cpuWorker.interrupt()` — flag set to `true`|
|~50ms+ε|Loop's next iteration checks `isInterrupted()`, sees `true`, exits loop, prints final count|

**Scenario 2 (Sleepy worker):**

|Time|Event|
|---|---|
|0ms|`sleepyWorker` starts, enters `sleep(10000)`|
|500ms|`main` calls `sleepyWorker.interrupt()`|
|500ms|`sleep()` immediately throws `InterruptedException` (flag cleared), caught, prints message, re-interrupts|

---

## Common Mistakes

1. **Assuming `interrupt()` forcibly stops a thread.**
    
    - It does not — it's purely a cooperative signal. A thread that never checks for it will simply ignore the interruption entirely and keep running.
2. **Silently swallowing `InterruptedException` (empty catch block).**
    
    - This discards the interruption signal entirely, potentially making the thread (and any calling code) unable to recognize that cancellation was requested — a very common, serious mistake in production code.
3. **Confusing `isInterrupted()` (instance method, doesn't clear flag) with `Thread.interrupted()` (static method, clears flag).**
    
    - Calling the wrong one can lead to either missing an interruption (if you keep clearing it accidentally) or repeatedly seeing a stale interruption you already should have handled.
4. **Writing tight, CPU-bound loops with no interruption check at all.**
    
    - Such a loop can never be gracefully cancelled via standard interruption — it will run to completion regardless of any `.interrupt()` calls, making it effectively "uncancellable" (this directly connects to the cancellation limitations discussed in [[07 - Future]]).

---

## Production Usage

- **`Future.cancel(true)`** (see [[07 - Future]]) works by internally calling `.interrupt()` on the executing thread — this is exactly why cancellation only works reliably for tasks that properly check/respond to interruption.
- **Graceful application shutdown**: Many production systems use interruption as part of their shutdown sequence — signaling background worker threads to stop their current loop and clean up, rather than abruptly killing them.
- **Timeout-based cancellation**: A common pattern combines a timer thread with `.interrupt()` calls to enforce a maximum execution time on a task, cooperatively asking it to stop if it runs too long.
- **`ExecutorService.shutdownNow()`**: Internally uses interruption to attempt to stop all actively executing tasks in a thread pool — again, this only reliably works for tasks written to check/respond to interruption (see [[46 - ExecutorService]]).

---

## Interview Questions

**Beginner:**

1. What does calling `thread.interrupt()` actually do?
2. Does `interrupt()` forcibly stop a thread?
3. What exception can be thrown as a result of interruption?

**Intermediate:** 4. What's the difference between `someThread.isInterrupted()` and the static `Thread.interrupted()`? 5. Why should you generally re-call `Thread.currentThread().interrupt()` inside a caught `InterruptedException` block? 6. What happens if `interrupt()` is called on a thread that's doing normal, non-blocking work and never checks its interrupted status?

**Advanced:** 7. How does `Future.cancel(true)` relate internally to thread interruption? 8. Why can a tight, CPU-bound loop with no interruption check be effectively "uncancellable" using standard Java mechanisms?

**Production-Level:** 9. How would you design a long-running background task to be properly cancellable via interruption? 10. Why is `Thread.stop()` deprecated, and how does standard interruption provide a safer alternative for requesting a thread to stop?

---

## Follow-up Questions

- If a thread is `BLOCKED` waiting to acquire a `synchronized` lock, can it be interrupted out of that wait? _(No — standard `synchronized` blocking is not interruptible; this is one of the key motivations for `ReentrantLock.lockInterruptibly()`, covered in [[32 - ReentrantLock]].)_
- Does interrupting a thread that has already terminated have any effect or throw an error? _(No error is thrown, but it has no meaningful effect, since there's no running code left to respond to the flag.)_
- Can a thread interrupt itself? _(Yes — `Thread.currentThread().interrupt()` is valid and commonly used precisely for the "re-interrupt after catching" pattern described in this note.)_

---

## Best Practices

- Always design long-running loops to periodically check `Thread.currentThread().isInterrupted()`.
- Never silently swallow `InterruptedException` — either handle it meaningfully or re-interrupt the thread to preserve the signal for calling code.
- Understand that standard `synchronized` blocking is not interruptible — use `ReentrantLock.lockInterruptibly()` (see [[32 - ReentrantLock]]) if you need cancellable lock acquisition.
- Treat interruption as the standard, idiomatic way to request cancellation in Java — avoid inventing custom "stop flags" when standard interruption already provides this mechanism cleanly.

---

## Performance Notes

- Interruption itself is a lightweight operation — setting a flag and, if applicable, waking a blocked thread has minimal overhead.
- The real "cost" consideration is architectural: code that doesn't check for interruption cannot be cancelled promptly, which can lead to resource waste (a task that should have stopped continuing to consume CPU/threads unnecessarily) in production systems that rely on timely cancellation.

---

## Comparison Table

|Aspect|`interrupt()`|`Thread.stop()` (deprecated)|
|---|---|---|
|Forces immediate termination?|No — cooperative only|Yes — but unsafely|
|Can leave shared state inconsistent?|No (thread controls its own cleanup)|Yes (can be stopped mid-operation, corrupting state)|
|Recommended in modern code?|Yes|No — deprecated, dangerous|
|Requires cooperating code?|Yes|No (which is exactly why it's dangerous)|

---

## Summary

- `thread.interrupt()` sets an internal flag and, if the thread is blocked in certain interruptible methods, wakes it with `InterruptedException` — it does **not** forcibly stop anything.
- Code must be written to **cooperate** with interruption by checking `isInterrupted()` in loops, or by properly handling `InterruptedException` from blocking calls.
- `isInterrupted()` (instance method) reads the flag without clearing it; `Thread.interrupted()` (static method) checks the current thread and clears the flag.
- Always re-interrupt (`Thread.currentThread().interrupt()`) after catching `InterruptedException` if you're not fully handling the cancellation right there, to preserve the signal for calling code.
- This mechanism underlies `Future.cancel(true)` and `ExecutorService.shutdownNow()` — both only work reliably for interruption-aware task code.

---

## Cheat Sheet

|API|Behavior|
|---|---|
|`thread.interrupt()`|Sets the interrupted flag; wakes the thread if blocked in an interruptible call|
|`someThread.isInterrupted()`|Instance method, reads flag, does NOT clear it|
|`Thread.interrupted()`|Static method, checks CURRENT thread, CLEARS the flag|
|`InterruptedException`|Thrown by `sleep()`, `wait()`, `join()`, and similar interruptible blocking calls|
|Best practice|Re-interrupt (`Thread.currentThread().interrupt()`) if not fully handling it locally|

---

## Keywords

Interrupt, InterruptedException, isInterrupted, Thread.interrupted, Cooperative Cancellation, Future.cancel, ExecutorService.shutdownNow

---

## Related Notes

[[09 - Thread Lifecycle]] [[18 - Sleep()]] [[20 - Join()]] [[07 - Future]] [[32 - ReentrantLock]] [[46 - ExecutorService]]

---

## Practice Exercises

**Easy:**

1. Create a thread with a loop checking `isInterrupted()`, interrupt it from the main thread after a short delay, and confirm it stops gracefully.
2. Create a thread that's sleeping, interrupt it, and confirm `InterruptedException` is caught and handled.

**Medium:** 3. Write a tight CPU-bound loop with **no** interruption check, attempt to interrupt it, and observe that it does NOT stop — demonstrating the "uncancellable" scenario firsthand. 4. Fix the loop from exercise 3 by adding a proper `isInterrupted()` check, and confirm it now responds correctly to interruption.

**Hard:** 5. Build a small task that performs a mix of CPU work and periodic `sleep()` calls, and confirm that interruption works correctly regardless of which "phase" the task is in when interrupted.

---

## Mini Project

**Project: Cancellable Background Worker**

Build a small console application that:

1. Runs a background worker thread simulating a long batch job — a loop of 1000 "units of work," each taking ~50ms (via `Thread.sleep(50)`), checking for interruption before each unit.
2. Provides a simple mechanism (e.g., the main thread sleeping 2 seconds, then calling `.interrupt()`) to request cancellation partway through.
3. Confirms the worker stops gracefully, printing how many units it completed before stopping, and demonstrates it did NOT complete all 1000 units.
4. As a bonus, add a **second**, deliberately "badly written" worker with no interruption checks, attempt to cancel it the same way, and confirm it runs to completion regardless — a clear, hands-on contrast.

**Goal:** Build a concrete, practical understanding of the difference between interruption-aware and interruption-unaware code, and why this distinction matters enormously for real-world cancellable task design.

---

## Common Interview Trap

**Trap Question:** _"If I call `thread.interrupt()` on a thread that's in the middle of a long, CPU-heavy `for` loop with no `sleep()` or `wait()` calls, will the loop stop?"_

The trap is assuming interruption has some inherent power to halt execution. The correct answer: **no, it will not stop**, unless the loop's own code explicitly checks `Thread.currentThread().isInterrupted()` somewhere within its body. Calling `.interrupt()` only sets a flag; it does not forcibly pause or terminate any code. A CPU-bound loop with no interruption check will run to completion entirely unaffected by any number of `.interrupt()` calls — this is precisely why writing interruption-aware code (regularly checking the flag in long-running loops) is considered an essential best practice for any task that might need to be cancelled.

---

## Frequently Asked Questions

**Q: Does interrupting a thread throw an exception immediately, in all cases?** A: No — only if the thread is currently blocked inside specific interruptible methods (`sleep()`, `wait()`, `join()`, and similar `java.util.concurrent` blocking calls). Otherwise, it just sets a flag that the thread's own code must check.

**Q: Is `synchronized` block/method blocking interruptible?** A: No — a thread `BLOCKED` waiting to enter a `synchronized` block cannot be interrupted out of that wait using standard interruption; `ReentrantLock.lockInterruptibly()` exists specifically to provide this capability (see [[32 - ReentrantLock]]).

**Q: What's the correct thing to do inside a `catch (InterruptedException e)` block if you're not going to fully stop the thread right there?** A: Call `Thread.currentThread().interrupt()` to re-set the flag, preserving the interruption signal for any code further up the call stack that might also need to be aware of it.

**Q: Does `Thread.stop()` still exist in modern Java?** A: It technically still exists but is deprecated and strongly discouraged, since it can forcibly terminate a thread mid-operation, potentially leaving shared objects in a corrupted, inconsistent state.

---

## Revision Checklist

- [ ] I understand that `interrupt()` is cooperative, not forceful.
- [ ] I know the difference between `isInterrupted()` and `Thread.interrupted()`.
- [ ] I understand why re-interrupting after catching `InterruptedException` is best practice.
- [ ] I know that standard `synchronized` blocking is not interruptible.
- [ ] I understand how `Future.cancel(true)` and `ExecutorService.shutdownNow()` rely on this mechanism.

---

## Flashcards

**Q:** Does `thread.interrupt()` forcibly stop a thread? **A:** No — it's a cooperative signal; the thread's own code must check for and respond to it.

**Q:** What's the difference between `isInterrupted()` and `Thread.interrupted()`? **A:** `isInterrupted()` is an instance method that reads the flag without clearing it; `Thread.interrupted()` is a static method that checks the current thread and clears the flag.

**Q:** Why should you re-call `Thread.currentThread().interrupt()` after catching `InterruptedException`? **A:** To preserve the interruption signal for any calling code further up the stack, since the flag was cleared when the exception was thrown.

**Q:** Can a thread `BLOCKED` on a `synchronized` lock be interrupted out of that wait? **A:** No — use `ReentrantLock.lockInterruptibly()` if you need that capability.

**Q:** What happens if a CPU-bound loop never checks `isInterrupted()`? **A:** It cannot be gracefully cancelled via interruption — it will run to completion regardless of any `.interrupt()` calls.

---

## References

- Oracle Java Tutorials — Interrupts: https://docs.oracle.com/javase/tutorial/essential/concurrency/interrupt.html
- `java.lang.Thread` API Documentation (`interrupt`, `isInterrupted`, `interrupted`)
- `java.util.concurrent.Future` API Documentation (`cancel`)