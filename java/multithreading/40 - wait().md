# wait()

## Objective

By the end of this note, you will understand:

- Exactly what `Object.wait()` does, at a precise mechanical level
- Why it must be called from within a `synchronized` block, and what happens if it isn't
- Why it releases the lock (unlike `sleep()`) and exactly when it re-acquires it
- The mandatory `while`-loop pattern and why skipping it is a real bug, not just a style preference

This note opens **Section 7: Inter-Thread Communication**, formalizing the mechanism that powered the Producer-Consumer implementation in [[39 - Producer Consumer]].

---

## Why should I learn this?

You've already used `wait()` in [[39 - Producer Consumer]], but this note isolates it completely to nail down its **exact contract** — a frequent, detailed interview topic. Getting any part of this wrong (calling it outside a synchronized block, using `if` instead of `while`, confusing it with `sleep()`) is one of the most common sources of subtle concurrency bugs among developers who've only seen `wait()` used, without understanding precisely what it guarantees.

---

## Beginner Explanation

`wait()` is a method every object has (inherited from `Object`), and it means: **"I'm currently inside a synchronized block on this object, but there's nothing useful for me to do right now — release my hold on this object's lock, and put me to sleep until someone tells me something has changed."**

Crucially: while waiting, this thread steps completely out of the way, releasing the lock so **other** threads can come in, do their work (which might be exactly the thing this waiting thread was waiting for), and then wake it back up.

---

## Real World Analogy

**Hospital Analogy**

Imagine a nurse sitting in a patient's room, specifically waiting for lab results to arrive before proceeding with treatment. Rather than pacing anxiously in the room, blocking the door and preventing anyone else from entering (this would be like `sleep()`, holding the lock the whole time), the nurse steps **out** of the room entirely, telling the front desk, "please page me the moment the lab results are ready" — freeing up the room for other staff to use in the meantime. When paged, the nurse returns, re-enters the room (re-acquires the "lock" on that space), and — importantly — **double-checks** that the lab results have actually arrived (rather than assuming the page was necessarily about their specific situation) before proceeding.

---

## Technical Explanation

### Method Signatures

```java
public final void wait() throws InterruptedException
public final void wait(long timeoutMillis) throws InterruptedException
public final void wait(long timeoutMillis, int nanos) throws InterruptedException
```

`wait()` is defined on `Object` itself (not `Thread`) — because it's fundamentally tied to an object's **monitor** (see [[23 - Monitor Lock]]), and every object has one.

### The Mandatory Precondition: Must Hold the Monitor

```java
synchronized (someObject) {
    someObject.wait(); // legal -- we hold someObject's monitor
}
```

```java
someObject.wait(); // ILLEGAL if not inside a synchronized(someObject) block
                    // throws IllegalMonitorStateException
```

Calling `wait()` **without** currently holding the target object's monitor throws `IllegalMonitorStateException` immediately — this makes sense once you understand what `wait()` actually does: it needs to **release** a lock, and you can't release a lock you don't hold.

### What `wait()` Actually Does, Precisely

1. The calling thread must currently hold `someObject`'s monitor.
2. `wait()` **atomically**: releases the monitor, and adds the thread to `someObject`'s internal **wait set**.
3. The thread's state becomes `WAITING` (or `TIMED_WAITING`, if a timeout was given) — see [[09 - Thread Lifecycle]].
4. The thread remains suspended until one of these happens:
    - Another thread calls `someObject.notify()` or `someObject.notifyAll()`.
    - The specified timeout elapses (for the timed overloads).
    - The thread is interrupted (throws `InterruptedException`).
5. Once woken, the thread must **re-acquire** `someObject`'s monitor before `wait()` can actually return — if another thread currently holds it, this waiting thread blocks (`BLOCKED`, not `WAITING`) until it becomes available.
6. Only after successfully re-acquiring the monitor does `wait()` return, and execution resumes on the line immediately following the `wait()` call.

### Why `while`, Not `if` — Formally

```java
synchronized (buffer) {
    while (buffer.isEmpty()) { // NOT if!
        buffer.wait();
    }
    // safe to proceed -- guaranteed non-empty HERE
}
```

Two distinct reasons make the `while` loop mandatory, not optional style:

1. **Spurious wakeups**: the JVM specification explicitly permits a waiting thread to wake up **without** any corresponding `notify()`/`notifyAll()` call — a rare but real, spec-permitted possibility. A `while` loop correctly re-verifies the condition regardless of why the thread woke up.
2. **Stolen conditions with multiple waiters**: even with a genuine `notifyAll()`, if **multiple** threads were waiting, only one of them "wins" the race to re-acquire the lock and act on the now-available condition first — by the time a **second** woken thread re-acquires the lock, the condition might already be false again (e.g., another consumer already took the one available item). Only a `while` loop correctly handles this.

### Why `wait()` Must Release the Lock (Contrast with `sleep()`)

If `wait()` did **not** release the lock, no other thread could ever enter the synchronized block to do the work that would eventually justify waking this thread up — the whole point of `wait()` is to step out of the way while remaining ready to be brought back in. This is precisely the property `Thread.sleep()` (see [[18 - Sleep()]]) lacks — `sleep()` holds onto any locks the entire time, which is exactly why `sleep()` is the wrong tool for "wait until some condition, set by another thread, becomes true."

---

## Internal Working

Step-by-step of `someObject.wait()`:

1. Precondition check: does the calling thread hold `someObject`'s monitor? If not, throw `IllegalMonitorStateException` immediately.
2. Atomically: release the monitor (its hold count drops to 0, becoming available to other threads) and register this thread in `someObject`'s internal wait set.
3. Suspend the thread (state: `WAITING`/`TIMED_WAITING`).
4. Upon `notify()`/`notifyAll()`/timeout/interrupt: remove the thread from the wait set (or mark it as eligible), and attempt to **re-acquire** the monitor.
5. If the monitor is currently held by another thread, this thread blocks (state: `BLOCKED`) until it becomes free.
6. Once re-acquired, `wait()` returns, and the thread resumes execution immediately following the call — where the `while` loop re-checks the original condition.

---

## Memory Diagram

```
        Object monitor
   -----------------------------------
   |  owner: Thread A (before wait)    |
   |  wait set: []                     |
   -----------------------------------

   Thread A calls wait():
   -----------------------------------
   |  owner: null (RELEASED)           |
   |  wait set: [Thread A]             |
   -----------------------------------

   Thread B acquires monitor, does work, calls notifyAll():
   -----------------------------------
   |  owner: Thread B (still held)     |
   |  wait set: [] (Thread A removed,  |
   |             now trying to        |
   |             re-acquire)           |
   -----------------------------------

   Thread B releases (exits synchronized block):
   -----------------------------------
   |  owner: Thread A (re-acquired)    |
   -----------------------------------
   Thread A's wait() call finally returns
```

---

## Execution Flow

```
Thread A: synchronized(obj) { while(!condition) obj.wait(); ...proceed... }
        |
        v
Acquires obj's monitor
        |
        v
condition false -> calls obj.wait()
        |
        v
RELEASES monitor, suspended (WAITING)
        |
   (meanwhile, Thread B does work, makes condition true, calls obj.notifyAll())
        |
        v
Thread A wakes, tries to RE-ACQUIRE monitor
        |
        v
Re-acquired -> wait() returns -> while loop RE-CHECKS condition
        |
        v
condition now true -> proceeds past the loop
```

---

## Code Example

```java
public class WaitDemo {

    static final Object lock = new Object();
    static boolean dataReady = false;
    static String data = null;

    public static void main(String[] args) throws InterruptedException {

        Thread waiter = new Thread(() -> {
            synchronized (lock) {
                while (!dataReady) {
                    try {
                        System.out.println("Waiter: no data yet, calling wait()...");
                        lock.wait();
                        System.out.println("Waiter: woke up, re-checking condition...");
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
                System.out.println("Waiter: data is ready! Value: " + data);
            }
        }, "Waiter");

        Thread producer = new Thread(() -> {
            try {
                Thread.sleep(1000); // simulate preparing data
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            synchronized (lock) {
                data = "Hello from producer!";
                dataReady = true;
                System.out.println("Producer: data set, calling notifyAll()...");
                lock.notifyAll();
            }
        }, "Producer");

        waiter.start();
        Thread.sleep(100); // ensure waiter starts waiting first
        producer.start();

        waiter.join();
        producer.join();
    }
}
```

### Line-by-line explanation

1. `waiter` acquires `lock`, checks `dataReady` (false), and calls `lock.wait()` — releasing the lock and suspending.
2. `producer`, after simulating 1 second of "preparation," acquires `lock`, sets `data` and `dataReady`, then calls `lock.notifyAll()`.
3. `waiter` wakes up, **re-acquires** `lock` (uncontended here, since `producer` is about to release it as its synchronized block ends), and the `while` loop re-checks `dataReady` — now `true` — so it proceeds to print the data.

---

## Dry Run

```
Waiter: no data yet, calling wait()...
[... roughly 1 second passes ...]
Producer: data set, calling notifyAll()...
Waiter: woke up, re-checking condition...
Waiter: data is ready! Value: Hello from producer!
```

---

## Common Mistakes

1. **Calling `wait()` outside a `synchronized` block on the same object.**
    
    - Throws `IllegalMonitorStateException` immediately — this is a compile-time-legal but runtime-illegal mistake, easy to make when refactoring code.
2. **Using `if` instead of `while` to guard the wait condition.**
    
    - As formally explained above, this is unsafe against both spurious wakeups and multi-waiter "stolen condition" scenarios — always use `while`.
3. **Confusing `wait()` with `sleep()`.**
    
    - `wait()` releases the lock and requires being called within a synchronized block on the target object; `sleep()` does neither (see [[18 - Sleep()]]). Using `sleep()` where `wait()` is needed prevents other threads from ever making the progress being waited for.
4. **Forgetting that `wait()` throws a checked `InterruptedException`, and mishandling it.**
    
    - Like other interruptible blocking calls (see [[21 - Interrupt()]]), this must be handled thoughtfully — typically by breaking out of the wait loop and propagating the interruption, not silently swallowing it.

---

## Production Usage

- **Every hand-rolled Producer-Consumer implementation** (see [[39 - Producer Consumer]]) using intrinsic locking relies directly on `wait()`.
- **Custom thread-pool/worker implementations** (before reaching for the JDK's `ExecutorService`) historically used `wait()` for idle worker threads waiting for new tasks to appear in a shared queue.
- **Object pooling implementations** sometimes use `wait()` for threads waiting for a pooled object to become available, before returning it.
- **Most modern production code** now prefers `ReentrantLock` + `Condition` (see [[35 - Condition]]) or, even more commonly, ready-made JDK utilities like `BlockingQueue` (see [[43 - BlockingQueue]]) over hand-rolled `wait()`/`notify()` — but understanding `wait()` deeply remains essential foundational knowledge.

---

## Interview Questions

**Beginner:**

1. What does `Object.wait()` do?
2. What exception is thrown if you call `wait()` without holding the object's monitor?
3. Does `wait()` release the lock it's called with?

**Intermediate:** 4. Why must the wait condition always be checked in a `while` loop, not an `if` statement? 5. What is a "spurious wakeup"? 6. What state does a waiting thread show in `Thread.getState()`?

**Advanced:** 7. Explain, step by step, exactly what happens internally when `wait()` is called and later returns. 8. Why does a thread need to RE-ACQUIRE the monitor before `wait()` can return, and what state does it show while doing so if the monitor is currently held by someone else?

**Production-Level:** 9. Why do most modern production codebases prefer `Condition` or `BlockingQueue` over raw `wait()`/`notify()`? 10. Describe a subtle bug that could occur if a `while` loop were mistakenly replaced with an `if` statement in a multi-consumer scenario.

---

## Follow-up Questions

- Can `wait()` be called on any object, or only specially designated ones? _(Any object — every object inherits `wait()` from `Object`, since every object has a monitor; there's nothing special required beyond currently holding that object's lock.)_
- Does `wait()` release ALL locks the thread holds, or just the one associated with the object it's called on? _(Only the specific monitor associated with the object `wait()` is called on — if the thread holds other, unrelated locks simultaneously, those are NOT released.)_
- What happens if `notify()` is called but no thread is currently waiting? _(Nothing happens — the notification is simply "lost"; it does not queue up or affect any future `wait()` call made afterward, which is an important, sometimes-surprising detail.)_

---

## Best Practices

- Always call `wait()` inside a `synchronized` block on the exact same object.
- Always use a `while` loop to guard the waited-for condition, never an `if`.
- Always handle `InterruptedException` from `wait()` thoughtfully — typically by exiting the wait loop and propagating the interruption.
- Consider `ReentrantLock` + `Condition` (see [[35 - Condition]]) for more complex coordination needs involving multiple distinct wait conditions.

---

## Performance Notes

- `wait()` properly suspends the thread with zero CPU consumption while waiting — a well-behaved, efficient blocking mechanism, unlike a busy-spin loop.
- Using a single object's monitor (and thus a single wait-set) for multiple, logically distinct waiting conditions can lead to inefficient "wake everyone, most go back to sleep" patterns under `notifyAll()` — this is exactly the limitation `Condition` (see [[35 - Condition]]) addresses with multiple independent wait-sets.

---

## Comparison Table

|Aspect|`wait()`|`sleep()` (see [[18 - Sleep()]])|
|---|---|---|
|Releases the lock?|Yes|No|
|Requires being inside `synchronized`?|Yes (on the same object)|No|
|Woken by|`notify()`/`notifyAll()`, timeout, or interrupt|Timeout, or interrupt|
|Typical use|Waiting for another thread to change shared state|Simple pause, unrelated to other threads' actions|

---

## Summary

- `Object.wait()` releases the calling thread's hold on that object's monitor and suspends it until notified, timed out, or interrupted.
- It **must** be called while holding the object's monitor (inside a `synchronized` block on that object), or it throws `IllegalMonitorStateException`.
- The wait condition must always be re-checked in a `while` loop after waking, to guard against spurious wakeups and multi-waiter race conditions.
- Unlike `sleep()`, `wait()` releases the lock — this is precisely what allows other threads to make the progress the waiting thread is depending on.
- `wait()`/`notify()` is the intrinsic-lock foundation that `ReentrantLock`+`Condition` (see [[35 - Condition]]) improves upon with multiple, independent wait-sets.

---

## Cheat Sheet

|Rule|Detail|
|---|---|
|Must be called inside|`synchronized(obj) { obj.wait(); }`|
|Releases the lock?|Yes|
|Exception if called without lock|`IllegalMonitorStateException`|
|Guard condition with|`while`, never `if`|
|Woken by|`notify()`, `notifyAll()`, timeout, or interrupt|
|Resulting state|`WAITING` or `TIMED_WAITING`|

---

## Keywords

wait, IllegalMonitorStateException, Spurious Wakeup, Wait Set, Monitor, while loop, Object.wait

---

## Related Notes

[[23 - Monitor Lock]] [[39 - Producer Consumer]] [[41 - notify()]] [[42 - notifyAll()]] [[35 - Condition]] [[18 - Sleep()]]

---

## Practice Exercises

**Easy:**

1. Write a program with a thread calling `wait()` on a shared object, and another thread calling `notifyAll()` after a delay — confirm the waiting thread resumes correctly.
2. Attempt to call `wait()` on an object without holding its monitor, and confirm the `IllegalMonitorStateException` is thrown.

**Medium:** 3. Reproduce this note's `WaitDemo`, and confirm the exact output ordering matches the expected flow. 4. Modify the demo to use `if` instead of `while`, then engineer a scenario (e.g., using multiple waiter threads) where this causes a real, observable bug.

**Hard:** 5. Implement a small "single-use gate" class: multiple threads call `awaitOpen()` (using `wait()`), blocking until a separate thread calls `open()` (using `notifyAll()`), after which all waiting threads should proceed. Test with several waiting threads to confirm all are correctly released.

---

## Mini Project

**Project: Wait/Notify Visualizer**

Build a small console application that:

1. Creates a shared object and several "waiter" threads that call `wait()` on it, each printing a timestamped message when they start waiting and when they wake up.
2. Creates a "notifier" thread that, after a delay, calls `notifyAll()`.
3. Logs the exact sequence: which threads started waiting, when notification occurred, and the order (potentially non-deterministic) in which each waiter successfully re-acquired the lock and resumed.

**Goal:** Build direct, observable insight into the full `wait()`/`notifyAll()` lifecycle with multiple waiters — reinforcing exactly why the re-acquisition step matters and why ordering among woken threads isn't guaranteed.

---

## Common Interview Trap

**Trap Question:** _"After `notifyAll()` is called, do all previously-waiting threads immediately resume running in parallel?"_

The trap is assuming `notifyAll()` grants simultaneous, parallel resumption to every waiting thread. The correct answer: **no** — `notifyAll()` simply makes all waiting threads **eligible** to compete for re-acquiring the monitor; since a monitor can only be held by **one** thread at a time, the woken threads must still take turns re-acquiring it, one at a time, in whatever order the JVM/OS scheduler decides (no ordering guarantee). Each thread, upon successfully re-acquiring the lock, re-checks its `while` condition — and if another thread already consumed the relevant resource in the meantime, it correctly goes back to waiting rather than incorrectly proceeding. This is precisely why the `while` loop (not `if`) is mandatory even after a genuine `notifyAll()` call.

---

## Frequently Asked Questions

**Q: Is `wait()` defined on `Thread` or `Object`?** A: `Object` — every object has `wait()` (along with `notify()`/`notifyAll()`), since it's fundamentally about that object's monitor, not about threads themselves.

**Q: Can a thread call `wait()` on multiple different objects, one after another?** A: Yes — there's nothing preventing a thread from waiting on different objects sequentially, though it can only be genuinely waiting on one object's monitor at any given instant (since `wait()` is itself a blocking call).

**Q: Does `wait()` guarantee the condition is true immediately upon returning?** A: No — this is exactly why the `while` loop is mandatory; `wait()` returning only means the thread was notified (or timed out, or was interrupted) and successfully re-acquired the lock — it does NOT guarantee the underlying condition is still true at that exact moment.

**Q: What happens to a thread's held locks (other than the one used with `wait()`) while it's waiting?** A: They remain held — `wait()` only releases the specific monitor it's called on; any other locks the thread happens to hold are unaffected and remain held throughout the wait.

---

## Revision Checklist

- [ ] I understand that `wait()` must be called while holding the target object's monitor.
- [ ] I understand that `wait()` releases that specific monitor, unlike `sleep()`.
- [ ] I can explain both reasons (spurious wakeups, multi-waiter races) that make the `while` loop mandatory.
- [ ] I understand the re-acquisition step that must succeed before `wait()` returns.
- [ ] I know `wait()` throws checked `InterruptedException`, requiring proper handling.

---

## Flashcards

**Q:** What must be true for `wait()` to be called legally? **A:** The calling thread must currently hold the target object's monitor (i.e., be inside a `synchronized` block on it).

**Q:** Does `wait()` release the lock it's called with? **A:** Yes.

**Q:** Why must the wait condition be checked in a `while` loop? **A:** To guard against spurious wakeups and against another thread "stealing" the condition before this thread resumes.

**Q:** What must happen before `wait()` can actually return? **A:** The thread must successfully re-acquire the object's monitor.

**Q:** What exception is thrown if `wait()` is called without holding the monitor? **A:** `IllegalMonitorStateException`.

---

## References

- Oracle Java Tutorials — Guarded Blocks: https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html
- `java.lang.Object` API Documentation (`wait`, `notify`, `notifyAll`)
- JLS (Java Language Specification) — Chapter 17: Threads and Locks