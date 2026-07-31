# Deadlock

## Objective

By the end of this note, you will understand:

- What a deadlock is, precisely, and the four necessary conditions for one to occur
- How to construct a minimal, reproducible deadlock example
- How to detect deadlocks in production using thread dumps
- The standard prevention strategies used in real systems

This note opens **Section 6: Concurrency Problems** — having built the synchronization toolkit in Sections 4-5, we now examine the specific failure modes that toolkit can introduce if misused.

---

## Why should I learn this?

Deadlock is the most infamous concurrency bug — a program that doesn't crash, doesn't throw an exception, just **freezes forever**. Every Java engineer eventually either causes one in production or has to debug one someone else caused. Understanding the precise mechanics (not just "two threads waiting on each other") lets you both avoid introducing them and quickly diagnose them when they occur.

---

## Beginner Explanation

Imagine two people trying to pass each other in a narrow hallway, each stepping to their own left to let the other by — but because they mirror each other's movement, they end up blocking each other indefinitely, neither able to proceed, both waiting for the other to move first.

A **deadlock** in code is exactly this: two (or more) threads, each holding a resource (a lock) that the other needs, and each waiting for the other to release theirs first. Since neither will ever voluntarily give up what it's holding until it gets what it's waiting for, **both wait forever**.

---

## Real World Analogy

**Bank Analogy**

Imagine two people trying to transfer money between each other's accounts at the exact same time:

- Person A locks their own account (to safely read/update its balance) and then tries to lock Person B's account (to complete the transfer).
- Person B, simultaneously, locks their own account and then tries to lock Person A's account.
- Person A is now waiting for Person B's account to become free — but Person B is waiting for Person A's account, which Person A is still holding.
- **Neither will ever let go**, because neither has finished their operation yet — and neither can finish, because each is waiting on the other. This is a genuine, classic banking-software deadlock pattern, and it maps directly onto the code example below.

---

## Technical Explanation

### The Four Necessary Conditions for Deadlock (Coffman Conditions)

A deadlock can only occur if **all four** of these conditions hold simultaneously:

1. **Mutual Exclusion**: At least one resource must be held in a non-shareable mode (only one thread can hold it at a time) — true of any lock.
2. **Hold and Wait**: A thread holding at least one resource is waiting to acquire additional resources currently held by other threads.
3. **No Preemption**: Resources cannot be forcibly taken away from a thread — a thread must voluntarily release what it holds.
4. **Circular Wait**: There exists a circular chain of threads, each waiting for a resource held by the next thread in the chain.

Breaking **any one** of these four conditions prevents deadlock entirely — this is the theoretical basis for all standard deadlock prevention strategies.

### Minimal Deadlock Example

```java
class Account {
    private double balance;
    private final Object lock = new Object();

    void transfer(Account to, double amount) {
        synchronized (this.lock) {          // Step 1: lock MY account
            synchronized (to.lock) {         // Step 2: lock THEIR account
                this.balance -= amount;
                to.balance += amount;
            }
        }
    }
}
```

If Thread 1 calls `accountA.transfer(accountB, 100)` and, **at the same time**, Thread 2 calls `accountB.transfer(accountA, 50)`:

- Thread 1 locks `accountA`, then tries to lock `accountB`.
- Thread 2 locks `accountB`, then tries to lock `accountA`.
- Thread 1 waits forever for `accountB` (held by Thread 2).
- Thread 2 waits forever for `accountA` (held by Thread 1).
- **Deadlock.**

### The Standard Fix: Lock Ordering

The most common, reliable deadlock prevention strategy is to **always acquire locks in a consistent, global order**, regardless of the direction of the operation — this breaks the **circular wait** condition:

```java
void transfer(Account to, double amount) {
    Account first = this.id < to.id ? this : to;   // always lock the LOWER id first
    Account second = this.id < to.id ? to : this;

    synchronized (first.lock) {
        synchronized (second.lock) {
            this.balance -= amount;
            to.balance += amount;
        }
    }
}
```

With this fix, **both** Thread 1 and Thread 2 will always attempt to lock the accounts in the **same** order (say, lower account ID first) — regardless of which direction the transfer is happening — meaning one of them will always successfully acquire both locks before the other even gets the first one, eliminating the circular wait entirely.

---

## Internal Working

Step-by-step of how the deadlock forms:

1. Thread 1 acquires `accountA`'s monitor.
2. Thread 2 acquires `accountB`'s monitor (this can happen concurrently, since they're different locks).
3. Thread 1 attempts to acquire `accountB`'s monitor — blocks, since Thread 2 holds it.
4. Thread 2 attempts to acquire `accountA`'s monitor — blocks, since Thread 1 holds it.
5. Both threads are now permanently `BLOCKED` (see [[09 - Thread Lifecycle]], [[10 - Thread States]]), each waiting for a lock the other holds, with **no** possibility of either ever proceeding — since progress would require the other thread to release its lock first, which requires **it** to make progress, which requires the first thread to release its lock... an unbreakable cycle.

---

## Memory Diagram

```
   Thread 1                          Thread 2
   --------                          --------
   HOLDS: accountA's lock            HOLDS: accountB's lock
   WANTS: accountB's lock            WANTS: accountA's lock
       |                                  |
       v                                  v
   accountB's lock                   accountA's lock
   (held by Thread 2)                (held by Thread 1)

   Thread 1 -> waiting on -> Thread 2 -> waiting on -> Thread 1
              (a CIRCLE -- this is the deadlock)
```

---

## Execution Flow

```
Thread 1: transfer(accountA -> accountB)     Thread 2: transfer(accountB -> accountA)
        |                                             |
        v                                             v
Lock accountA                                  Lock accountB
        |                                             |
        v                                             v
Try to lock accountB -- BLOCKED               Try to lock accountA -- BLOCKED
   (Thread 2 holds it)                           (Thread 1 holds it)
        |                                             |
        +--------------- DEADLOCK ---------------------+
        (neither can ever proceed; both wait forever)
```

---

## Code Example

```java
public class DeadlockDemo {

    static class Account {
        final int id;
        double balance;
        final Object lock = new Object();

        Account(int id, double balance) {
            this.id = id;
            this.balance = balance;
        }

        // BUGGY VERSION -- causes deadlock
        void transferBuggy(Account to, double amount) {
            synchronized (this.lock) {
                System.out.println(Thread.currentThread().getName() + " locked account " + this.id);
                try { Thread.sleep(100); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                synchronized (to.lock) {
                    this.balance -= amount;
                    to.balance += amount;
                }
            }
        }

        // FIXED VERSION -- consistent lock ordering
        void transferFixed(Account to, double amount) {
            Account first = this.id < to.id ? this : to;
            Account second = this.id < to.id ? to : this;
            synchronized (first.lock) {
                synchronized (second.lock) {
                    this.balance -= amount;
                    to.balance += amount;
                }
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Account accountA = new Account(1, 1000);
        Account accountB = new Account(2, 1000);

        Thread t1 = new Thread(() -> accountA.transferBuggy(accountB, 100), "Thread-1");
        Thread t2 = new Thread(() -> accountB.transferBuggy(accountA, 50), "Thread-2");

        t1.start();
        t2.start();

        t1.join(3000); // wait up to 3 seconds
        t2.join(3000);

        if (t1.isAlive() || t2.isAlive()) {
            System.out.println("DEADLOCK DETECTED: threads are still alive after 3 seconds!");
        } else {
            System.out.println("Both transfers completed successfully.");
        }
    }
}
```

### Line-by-line explanation

1. `transferBuggy()` always locks `this` first, then `to` — creating the exact circular-wait risk described above when two transfers happen in opposite directions concurrently.
2. `t1` transfers from A to B; `t2` transfers from B to A, at nearly the same time — this is precisely the setup needed to trigger the deadlock.
3. Using `join(3000)` (a bounded wait, see [[20 - Join()]]) rather than an unbounded `join()` lets the program detect the likely deadlock and report it, rather than hanging forever itself.
4. **Fixing it**: replace `accountA.transferBuggy(accountB, 100)` and `accountB.transferBuggy(accountA, 50)` with `transferFixed()` calls, and the deadlock will no longer occur, since both transfers will consistently attempt to lock the lower-ID account first.

---

## Dry Run

**With the buggy version:**

```
Thread-1 locked account 1
Thread-2 locked account 2
[... program hangs here ...]
DEADLOCK DETECTED: threads are still alive after 3 seconds!
```

**With the fixed version:**

```
Both transfers completed successfully.
```

---

## Common Mistakes

1. **Acquiring multiple locks in inconsistent order across different code paths.**
    
    - This is, by far, the most common real-world cause of deadlocks — as demonstrated, even a single method can cause deadlock if called with "mirrored" arguments from different threads.
2. **Holding a lock while calling into external/unknown code (that might itself try to acquire a lock).**
    
    - If external code (a callback, a listener, a library method) unexpectedly tries to acquire a lock you're currently holding while waiting on something it holds, this can create a deadlock that's very difficult to predict from your code alone.
3. **Assuming deadlocks are rare/unlikely and not worth designing against.**
    
    - Since deadlocks depend on precise timing (both threads needing to reach their respective first lock at nearly the same time), a genuinely buggy lock-ordering pattern might work correctly 99.9% of the time in testing and still cause a production outage under real concurrent load.
4. **Not using thread dumps to diagnose a suspected deadlock, and instead guessing based on code inspection alone.**
    
    - Thread dumps (see [[10 - Thread States]]) explicitly detect and report deadlock cycles — always use this tool first when a production system appears to be hanging.

---

## Production Usage

- **Banking/financial transfer systems** (like this note's example) are a canonical, real-world deadlock risk category — any system involving "lock resource A, then lock resource B" operations across multiple resources needs careful lock-ordering discipline.
- **`jstack`-based deadlock detection** (see [[10 - Thread States]]) is a standard first response tool when a production Java service appears to be hung — its explicit "Found one Java-level deadlock" section directly identifies the cycle.
- **Code review checklists** at serious engineering organizations often specifically flag "does this method acquire more than one lock? If so, is the acquisition order globally consistent?" as a standard review question.
- **Static analysis tools** (some IDEs and dedicated tools) can flag potential lock-ordering inconsistencies across a codebase, though they can't catch every case — careful design remains essential.

---

## Interview Questions

**Beginner:**

1. What is a deadlock?
2. What are the four necessary conditions for a deadlock to occur (Coffman conditions)?
3. Give a simple, real-world analogy for a deadlock.

**Intermediate:** 4. What is the standard technique for preventing deadlocks in a multi-lock system? 5. Which of the four Coffman conditions does consistent lock ordering break? 6. How would you detect a deadlock in a running production Java application?

**Advanced:** 7. Walk through, step by step, exactly how two threads transferring money between the same two accounts (in opposite directions) can deadlock. 8. Why can a deadlock-prone piece of code work correctly 99% of the time in testing, yet still cause a production outage?

**Production-Level:** 9. Design a code review checklist item that would catch the kind of lock-ordering bug demonstrated in this note. 10. Besides consistent lock ordering, name another strategy for avoiding deadlocks (e.g., using `tryLock()` with a timeout), and explain how it breaks one of the four Coffman conditions.

---

## Follow-up Questions

- Can a deadlock occur with only ONE lock involved? _(No — deadlock fundamentally requires a circular wait among two or more resources; a single lock can cause a thread to wait indefinitely if never released, but that's a different problem — a lock leak/starvation, not a deadlock, which specifically requires a cycle.)_
- Does `ReentrantLock`'s `tryLock(timeout, unit)` help prevent deadlocks? _(Yes — if a thread can't acquire the second lock within a timeout, it can release the first lock and retry later, breaking the "hold and wait" condition rather than waiting forever.)_
- Is a deadlock the same thing as an infinite loop? _(No — in a deadlock, threads aren't executing any code at all; they're `BLOCKED`, consuming no CPU, just permanently suspended waiting for a lock that will never become available. An infinite loop, by contrast, actively consumes CPU cycles continuously.)_

---

## Best Practices

- Always acquire multiple locks in a **globally consistent order** across your entire codebase (e.g., by a stable ID, hash code, or some other well-defined ordering).
- Consider using `tryLock(timeout, unit)` (see [[32 - ReentrantLock]]) instead of unconditional blocking acquisition when deadlock risk is a genuine concern, allowing a thread to back off and retry rather than wait forever.
- Avoid holding a lock while calling into external or unpredictable code.
- Use thread dumps as your first diagnostic tool whenever a production system appears hung — don't guess.

---

## Performance Notes

- Deadlock itself has no "performance cost" in the traditional sense — the affected threads simply stop making progress entirely, consuming zero CPU while blocked, but also producing zero useful output, effectively taking down whatever functionality depends on them.
- Prevention strategies like consistent lock ordering have essentially no performance cost — they're purely a discipline/design change, not a runtime overhead.
- `tryLock()`-based avoidance strategies do add some complexity (retry logic) but can provide better resilience in systems where lock-ordering discipline is hard to guarantee across a large, evolving codebase.

---

## Comparison Table

|Concept|Deadlock|Livelock (see [[37 - Livelock]])|Starvation (see [[38 - Starvation]])|
|---|---|---|---|
|Threads consuming CPU?|No — fully blocked|Yes — actively "working" but making no progress|Possibly — waiting indefinitely, but not necessarily in a cycle|
|Requires a cycle?|Yes|Not necessarily|No|
|Fixed by lock ordering?|Yes|Not directly|Not directly|

---

## Summary

- A deadlock occurs when two or more threads are each waiting for a resource held by another, forming a circular wait with no possibility of progress.
- It requires all four Coffman conditions simultaneously: mutual exclusion, hold-and-wait, no preemption, and circular wait.
- The standard, most reliable prevention technique is **consistent lock ordering** across all code paths that acquire multiple locks.
- `tryLock()` with a timeout is an alternative strategy that breaks the "hold and wait" condition by allowing a thread to back off rather than wait forever.
- Thread dumps (`jstack`) explicitly detect and report deadlock cycles — always use this tool to diagnose a suspected production deadlock.

---

## Cheat Sheet

|Coffman Condition|Broken By|
|---|---|
|Mutual Exclusion|(Usually can't/shouldn't break this — it's often required for correctness)|
|Hold and Wait|`tryLock()` with timeout, releasing and retrying|
|No Preemption|(Rarely broken in typical Java code — would require forcibly revoking locks)|
|Circular Wait|Consistent, global lock ordering|

---

## Keywords

Deadlock, Coffman Conditions, Circular Wait, Lock Ordering, tryLock, jstack, Hold and Wait

---

## Related Notes

[[10 - Thread States]] [[32 - ReentrantLock]] [[37 - Livelock]] [[38 - Starvation]] [[69 - Thread Dump Analysis]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's buggy transfer deadlock and confirm both threads hang using bounded `join()` detection.
2. Fix it using consistent lock ordering (`transferFixed()`) and confirm the deadlock no longer occurs.

**Medium:** 3. Take a `jstack` dump while the buggy version is hung, and locate the "Found one Java-level deadlock" section, identifying which threads and locks are involved. 4. Reimplement the fix using `tryLock(timeout, unit)` instead of consistent ordering, releasing and retrying if the second lock can't be acquired in time.

**Hard:** 5. Construct a 3-thread, 3-lock deadlock scenario (A waits for B, B waits for C, C waits for A) and confirm it also produces a genuine deadlock, then fix it using consistent lock ordering across all three resources.

---

## Mini Project

**Project: Deadlock Detector and Fixer**

Build a small console application that:

1. Implements the buggy bank transfer scenario from this note, running many concurrent, randomly-directed transfers between a set of 5 accounts.
2. Detects deadlock using bounded `join()` timeouts, and additionally captures a programmatic thread dump (via `ThreadMXBean.findDeadlockedThreads()`, see [[10 - Thread States]]) to confirm and report the deadlock precisely.
3. Reimplements the transfer logic using consistent lock ordering, re-runs the same random-transfer simulation, and confirms zero deadlocks occur, even after many runs.

**Goal:** Build a complete, realistic, hands-on deadlock reproduction-and-fix cycle — including using the exact same detection tools (`ThreadMXBean`) that real production engineers use.

---

## Common Interview Trap

**Trap Question:** _"If two threads both try to acquire the same two locks, but they DON'T do so in opposite orders, can a deadlock still occur?"_

The trap is assuming deadlock is only about "opposite order" scenarios in a simple two-thread, two-lock case. The correct, deeper answer: **deadlock requires a circular wait**, but this circularity can involve more than two threads and more than two locks in more complex, multi-step chains (e.g., Thread A waits for a lock held by Thread B, which waits for a lock held by Thread C, which waits for a lock held by Thread A) — the "opposite order" framing is just the simplest, most common **two**-thread illustration. In real systems with many locks and many code paths, consistent **global** lock ordering (not just "don't reverse the order between two specific methods") is the only reliable way to guarantee no circular wait can ever form, regardless of how many threads or locks end up involved in any given execution.

---

## Frequently Asked Questions

**Q: Can a deadlock resolve itself over time, given enough patience?** A: No — by definition, none of the involved threads can make any progress without external intervention (like restarting the affected threads/process), since each is waiting for something only another deadlocked thread could provide.

**Q: Does using `synchronized` versus `ReentrantLock` change the fundamental deadlock risk?** A: No — deadlock is a logical/design issue about the order in which locks are acquired, not something inherent to either specific locking mechanism; both can deadlock if used with inconsistent ordering, and both can be protected against using the same lock-ordering discipline (with `ReentrantLock` additionally offering `tryLock()`-based avoidance as an alternative strategy).

**Q: Are deadlocks unique to Java, or a general concurrent programming issue?** A: General — deadlock is a fundamental concept in operating systems and concurrent programming across all languages; Java's specific tools for detecting (`jstack`, `ThreadMXBean`) and preventing (lock ordering, `tryLock()`) it are Java-specific, but the underlying problem is universal.

**Q: Can database transactions deadlock in a way similar to Java thread locks?** A: Yes — database-level deadlocks (e.g., two transactions each holding a row lock the other needs) follow the exact same Coffman conditions and are typically resolved automatically by the database engine, which detects the cycle and forcibly rolls back one of the transactions — a different mitigation strategy (preemption) than what's typically used at the application thread level in Java.

---

## Revision Checklist

- [ ] I can state the four Coffman conditions required for deadlock.
- [ ] I can construct a minimal, reproducible deadlock example.
- [ ] I understand why consistent lock ordering prevents deadlock (breaks circular wait).
- [ ] I know how to detect a deadlock using thread dumps or `ThreadMXBean`.
- [ ] I understand that deadlock requires a cycle, distinguishing it from simple starvation or an infinite loop.

---

## Flashcards

**Q:** What are the four Coffman conditions required for deadlock? **A:** Mutual exclusion, hold and wait, no preemption, circular wait.

**Q:** What is the standard technique for preventing deadlock across multiple locks? **A:** Consistent, global lock ordering.

**Q:** Which Coffman condition does consistent lock ordering break? **A:** Circular wait.

**Q:** How can you detect a deadlock in a running Java application? **A:** Using a thread dump (`jstack`), which explicitly reports detected deadlock cycles, or programmatically via `ThreadMXBean.findDeadlockedThreads()`.

**Q:** Do deadlocked threads consume CPU? **A:** No — they are fully `BLOCKED`, consuming no CPU, simply making no progress forever.

---

## References

- Oracle Java Tutorials — Deadlock: https://docs.oracle.com/javase/tutorial/essential/concurrency/deadlock.html
- Coffman, E.G. et al. (1971) — "System Deadlocks" (foundational paper defining the four necessary conditions)
- `java.lang.management.ThreadMXBean` API Documentation (`findDeadlockedThreads`)