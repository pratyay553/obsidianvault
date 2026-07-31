# Condition

## Objective

By the end of this note, you will understand:

- What the `Condition` interface is and the problem it solves
- How it relates to (and improves upon) intrinsic `wait()`/`notify()`
- The key advantage of multiple, independent condition queues per lock
- How to use `Condition` correctly with `ReentrantLock`

This note closes out **Section 5: Advanced Locks**, and directly sets up [[39 - Producer Consumer]], [[40 - wait()]], [[41 - notify()]], and [[42 - notifyAll()]] — `Condition` is the `ReentrantLock`-based equivalent of intrinsic wait/notify, with genuine improvements.

---

## Why should I learn this?

Every object's intrinsic monitor (see [[23 - Monitor Lock]]) has exactly **one** built-in wait-set — meaning `wait()`/`notify()` on that object can only coordinate around a single, undifferentiated condition. Many real coordination problems actually involve **multiple distinct conditions** (e.g., a bounded buffer needs to wake up "waiting to add" threads differently from "waiting to remove" threads), and cramming all of this into a single intrinsic wait-set is awkward and inefficient. `Condition`, paired with `ReentrantLock`, directly solves this by letting you create **multiple, independent** condition queues per lock.

---

## Beginner Explanation

Imagine a single waiting room with one bell that everyone shares, regardless of what they're actually waiting for — patients waiting for an X-ray, patients waiting for lab results, and patients waiting for discharge paperwork are all mixed together in this one room, waiting for the same one bell to ring. When the bell rings, **everyone** wakes up and checks if it's relevant to them — even though most of the time it isn't, since the bell doesn't distinguish between the different reasons people are waiting.

`Condition` objects let you set up **separate waiting rooms with separate bells** — one for X-ray patients, one for lab results, one for discharge — so that when a nurse rings the "X-ray ready" bell, only the X-ray-waiting patients wake up, and everyone else stays comfortably asleep, undisturbed.

---

## Real World Analogy

**Restaurant Analogy**

Imagine a restaurant with a single "your table is ready" buzzer shared by every waiting party, regardless of party size or seating section — a party of 2 waiting for a small table and a party of 8 waiting for a large table all share the exact same buzzer. Every time the buzzer goes off, **every** waiting party checks their status, even though the vast majority of the time it's irrelevant to them.

A better-designed restaurant might have **separate buzzer systems**: one specifically for small tables, one for large tables. When a small table opens up, only the "small table" buzzer sounds, and only those specific parties need to check in — large-table parties remain undisturbed. This is exactly the improvement `Condition` provides over a single intrinsic wait-set: multiple, independent, targeted "buzzer systems" tied to the same overall lock.

---

## Technical Explanation

### Creating and Using a `Condition`

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

Lock lock = new ReentrantLock();
Condition notFull = lock.newCondition();
Condition notEmpty = lock.newCondition();
```

A `Condition` object is always created **from** a specific `Lock` instance via `newCondition()` — it's intrinsically tied to that lock, since waiting on a condition requires releasing (and later re-acquiring) that specific lock, exactly parallel to how intrinsic `wait()` releases the monitor it's called within.

### The Core Methods

```java
condition.await();          // like Object.wait() -- releases the lock, waits indefinitely
condition.await(2, TimeUnit.SECONDS); // like Object.wait(timeout)
condition.signal();         // like Object.notify() -- wakes ONE waiting thread on THIS condition
condition.signalAll();      // like Object.notifyAll() -- wakes ALL waiting threads on THIS condition
```

**Critical rule**: exactly like `wait()`/`notify()`, you must hold the associated **lock** before calling any of these methods — calling `condition.await()` (or `signal()`) without holding the lock throws `IllegalMonitorStateException`.

### Full Example: Bounded Buffer with Two Separate Conditions

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await(); // wait specifically for "not full" -- releases lock while waiting
            }
            queue.add(item);
            notEmpty.signal(); // wake ONE thread waiting specifically for "not empty"
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await(); // wait specifically for "not empty"
            }
            T item = queue.poll();
            notFull.signal(); // wake ONE thread waiting specifically for "not full"
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**This is the key advantage over intrinsic locking**: with a plain `synchronized`/`wait()`/`notify()` approach, you'd have only **one** wait-set for this entire class, meaning a `notifyAll()` after adding an item would wake up **both** producer threads (waiting for "not full," irrelevant right now) and consumer threads (waiting for "not empty," relevant right now) — forcing all of them to wake up, re-check their condition, and most go back to sleep, which is wasteful. With two separate `Condition` objects, you can precisely signal **only** the relevant waiting threads.

### Why `while`, Not `if`, for the Wait Condition

Just like intrinsic `wait()` (see [[40 - wait()]]), you must **always** re-check the condition in a `while` loop after waking up, not assume it's now true just because you were signaled — this protects against spurious wakeups and against another thread "stealing" the condition (e.g., another consumer draining the queue before this thread gets scheduled after waking).

---

## Internal Working

Step-by-step of `condition.await()`:

1. The calling thread must currently hold the associated `Lock`.
2. `await()` **releases** the lock and adds the thread to this specific `Condition`'s internal wait queue (separate from any other `Condition`'s wait queue on the same lock).
3. The thread is suspended (state: `WAITING` or `TIMED_WAITING`).
4. When another thread calls `signal()` (or `signalAll()`) on this **same** `Condition` object, one (or all) waiting threads on it are moved to a state where they can attempt to **re-acquire** the lock.
5. Once the lock is available and re-acquired, `await()` returns, and the thread resumes execution — typically re-checking its `while` condition immediately.

---

## Memory Diagram

```
        ReentrantLock
   -----------------------------------
   |  Condition "notFull"              |
   |    wait queue: [Producer-1]        |
   -----------------------------------
   |  Condition "notEmpty"              |
   |    wait queue: [Consumer-1, C-2]   |
   -----------------------------------

   notFull.signal() -> wakes ONLY Producer-1
   (Consumer-1 and Consumer-2 remain undisturbed)
```

---

## Execution Flow

```
Producer calls put(item)
        |
        v
Acquires lock
        |
        v
Queue full? -> notFull.await() (releases lock, waits on "notFull" queue specifically)
        |
   (meanwhile, a consumer calls take(), removes an item)
        |
        v
Consumer calls notFull.signal() -- wakes ONLY producer threads waiting on "notFull"
        |
        v
Producer re-acquires lock, re-checks while condition, proceeds to add item
        |
        v
Producer calls notEmpty.signal() -- wakes ONLY consumer threads waiting on "notEmpty"
```

---

## Code Example

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionDemo {

    static class BoundedBuffer {
        private final Queue<Integer> queue = new LinkedList<>();
        private final int capacity = 3;
        private final Lock lock = new ReentrantLock();
        private final Condition notFull = lock.newCondition();
        private final Condition notEmpty = lock.newCondition();

        void put(int item) throws InterruptedException {
            lock.lock();
            try {
                while (queue.size() == capacity) {
                    System.out.println("Producer waiting: buffer full.");
                    notFull.await();
                }
                queue.add(item);
                System.out.println("Produced: " + item + " (buffer size: " + queue.size() + ")");
                notEmpty.signal();
            } finally {
                lock.unlock();
            }
        }

        int take() throws InterruptedException {
            lock.lock();
            try {
                while (queue.isEmpty()) {
                    System.out.println("Consumer waiting: buffer empty.");
                    notEmpty.await();
                }
                int item = queue.poll();
                System.out.println("Consumed: " + item + " (buffer size: " + queue.size() + ")");
                notFull.signal();
                return item;
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        BoundedBuffer buffer = new BoundedBuffer();

        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 6; i++) {
                    buffer.put(i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 6; i++) {
                    Thread.sleep(150); // consume slower than production
                    buffer.take();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer");

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();
    }
}
```

### Line-by-line explanation

1. `BoundedBuffer` uses two separate `Condition`s — `notFull` for producers, `notEmpty` for consumers — sharing the same underlying `ReentrantLock`.
2. Since the buffer's capacity is 3, and the consumer is deliberately slower, the producer will fill the buffer and then call `notFull.await()`, printing "buffer full" — waiting specifically on the `notFull` condition.
3. When the consumer calls `take()` and calls `notFull.signal()`, only threads waiting on `notFull` (the producer) are woken — this precise targeting is the core benefit demonstrated here.

---

## Dry Run

Example output (exact interleaving will vary slightly):

```
Produced: 1 (buffer size: 1)
Produced: 2 (buffer size: 2)
Produced: 3 (buffer size: 3)
Producer waiting: buffer full.
Consumed: 1 (buffer size: 2)
Produced: 4 (buffer size: 3)
Producer waiting: buffer full.
Consumed: 2 (buffer size: 2)
Produced: 5 (buffer size: 3)
Producer waiting: buffer full.
Consumed: 3 (buffer size: 2)
Produced: 6 (buffer size: 3)
Consumed: 4 (buffer size: 2)
Consumed: 5 (buffer size: 1)
Consumed: 6 (buffer size: 0)
```

---

## Common Mistakes

1. **Calling `await()`/`signal()` without holding the associated lock.**
    
    - Throws `IllegalMonitorStateException` — just like intrinsic `wait()`/`notify()` requiring the monitor to be held.
2. **Using `if` instead of `while` to check the waiting condition.**
    
    - Exactly like intrinsic `wait()`, spurious wakeups and race conditions between multiple waiters mean you must always re-check the condition in a loop after waking.
3. **Using a single `Condition` when multiple, logically distinct wait conditions would be clearer and more efficient.**
    
    - This defeats the primary advantage `Condition` offers over intrinsic wait/notify — if you're only ever going to use one condition per lock, intrinsic `wait()`/`notify()` might be simpler.
4. **Forgetting that a `Condition` is tied to a specific `Lock` instance and cannot be used independently of it.**
    
    - You cannot call `condition.await()` without having acquired the specific `Lock` that created that `Condition` object.

---

## Production Usage

- **Custom blocking queue implementations** (conceptually similar to what `BlockingQueue`'s own JDK implementations do internally, see [[43 - BlockingQueue]]) commonly use paired `Condition`s exactly like this note's bounded buffer example.
- **Custom connection pool implementations** sometimes use `Condition` to separately coordinate "waiting for an available connection" versus other distinct wait scenarios.
- **Producer-consumer pipelines** in data processing systems, where cleanly separating "buffer full" and "buffer empty" wait conditions improves both code clarity and signaling efficiency compared to a single shared wait-set.
- **In most everyday application code**, developers reach for the JDK's already-implemented `BlockingQueue` (see [[43 - BlockingQueue]]) rather than hand-rolling `Condition`-based buffers — but understanding this note's pattern clarifies exactly what's happening inside those higher-level utilities.

---

## Interview Questions

**Beginner:**

1. What is a `Condition`, and how do you create one?
2. What is the key advantage of `Condition` over intrinsic `wait()`/`notify()`?
3. What exception is thrown if you call `condition.await()` without holding the associated lock?

**Intermediate:** 4. How does `signal()` differ from `signalAll()`? 5. Why must the wait condition always be checked in a `while` loop, not an `if` statement? 6. Can multiple `Condition` objects be created from a single `Lock`?

**Advanced:** 7. Explain, using the bounded buffer example, exactly why having two separate `Condition`s is more efficient than a single shared wait-set. 8. Why is a `Condition` object always tied to the specific `Lock` instance that created it?

**Production-Level:** 9. Describe a real scenario where separating wait conditions using multiple `Condition` objects would meaningfully improve performance over a single intrinsic wait-set. 10. Why might you choose to use the JDK's `BlockingQueue` instead of hand-rolling a `Condition`-based buffer in most production code?

---

## Follow-up Questions

- Is `Condition.await()` interruptible, like `lockInterruptibly()`? _(Yes — `await()` throws `InterruptedException` if the waiting thread is interrupted, similar to intrinsic `wait()`.)_
- Does a `Condition` support a version of `await()` with a timeout? _(Yes — `await(long time, TimeUnit unit)`, directly analogous to `Object.wait(timeout)`.)_
- Can a single `Lock` have more than two `Condition` objects? _(Yes — there's no fixed limit; you can create as many distinct conditions as your coordination logic genuinely needs.)_

---

## Best Practices

- Use multiple `Condition` objects when your coordination logic genuinely involves multiple, distinct wait scenarios (like a bounded buffer's full/empty states) — this improves both clarity and signaling efficiency.
- Always check the wait condition in a `while` loop, exactly as with intrinsic `wait()`.
- Prefer the JDK's existing `BlockingQueue` implementations for standard producer-consumer needs rather than reimplementing this pattern from scratch, unless you have a genuinely custom requirement.
- Remember that `Condition` requires `ReentrantLock` (or another `Lock` implementation) — it cannot be used with intrinsic `synchronized` blocks.

---

## Performance Notes

- Using targeted `signal()` calls on specific `Condition`s (rather than `notifyAll()` waking every waiter indiscriminately) reduces unnecessary wakeups and re-checks, improving efficiency in systems with multiple distinct wait scenarios.
- The performance benefit of separate conditions grows with the number of concurrently waiting threads and the degree to which their wait reasons genuinely differ — for simple, single-condition scenarios, intrinsic `wait()`/`notify()` may be simpler with negligible performance difference.

---

## Comparison Table

|Aspect|Intrinsic `wait()`/`notify()`|`Condition`|
|---|---|---|
|Number of wait-sets per lock|Exactly one|Multiple (one per `Condition` object created)|
|Requires|`synchronized` block|`ReentrantLock` (or other `Lock`)|
|Timeout support|Yes (`wait(ms)`)|Yes (`await(time, unit)`)|
|Interruptible|Yes|Yes|
|Targeted signaling|No — `notify()` picks an arbitrary waiter from the single set|Yes — `signal()` targets a specific condition's waiters only|

---

## Summary

- `Condition` is `ReentrantLock`'s equivalent of intrinsic `wait()`/`notify()`, obtained via `lock.newCondition()`.
- Its key advantage is supporting **multiple, independent** wait-sets per lock, allowing precise, targeted signaling (e.g., separate "not full" and "not empty" conditions for a bounded buffer).
- Core methods: `await()` (like `wait()`), `signal()` (like `notify()`), `signalAll()` (like `notifyAll()`) — all requiring the associated lock to be held.
- Always re-check the wait condition in a `while` loop after waking, exactly as with intrinsic wait/notify.
- Most production code uses higher-level utilities like `BlockingQueue` rather than hand-rolling `Condition`-based coordination, but understanding this pattern clarifies how those utilities work internally.

---

## Cheat Sheet

|Intrinsic|`Condition` Equivalent|
|---|---|
|`synchronized(lock) { ... }`|`lock.lock(); try { ... } finally { lock.unlock(); }`|
|`obj.wait()`|`condition.await()`|
|`obj.notify()`|`condition.signal()`|
|`obj.notifyAll()`|`condition.signalAll()`|
|One wait-set per object|Multiple `Condition`s per `Lock`|

---

## Keywords

Condition, await, signal, signalAll, newCondition, Bounded Buffer, Targeted Signaling, ReentrantLock

---

## Related Notes

[[32 - ReentrantLock]] [[39 - Producer Consumer]] [[40 - wait()]] [[41 - notify()]] [[42 - notifyAll()]] [[43 - BlockingQueue]]

---

## Practice Exercises

**Easy:**

1. Create a `ReentrantLock` with a single `Condition`, and implement a simple "wait until a flag becomes true" coordination between two threads.
2. Rewrite the same logic using intrinsic `wait()`/`notify()` instead, and compare the code structure.

**Medium:** 3. Reproduce this note's bounded buffer example, and confirm the producer correctly waits when full and the consumer correctly waits when empty. 4. Extend the bounded buffer with a third `Condition` for some additional coordination need (e.g., "buffer is exactly half full") and demonstrate it being signaled appropriately.

**Hard:** 5. Build a simple thread-safe "parking garage" simulation: cars (threads) wait on a "spotAvailable" condition to enter, and departing cars signal it; separately, a "garageFull" condition could notify an attendant thread when capacity is reached — demonstrating multiple independent conditions coordinating a realistic scenario.

---

## Mini Project

**Project: Multi-Condition Task Queue**

Build a small console application that:

1. Implements a task queue with a maximum size, using `ReentrantLock` and two `Condition`s (`notFull`, `notEmpty`), similar to this note's bounded buffer.
2. Adds a third `Condition`, `isEmpty`, specifically signaled when the queue transitions to fully empty, used by a separate "monitoring" thread that logs a message whenever this happens (distinct from the regular consumer's `notEmpty` waiting).
3. Runs multiple producer and consumer threads against this queue, along with the monitoring thread, and confirms all three conditions coordinate correctly and efficiently.

**Goal:** Build hands-on comfort working with more than two `Condition` objects on a single lock, reinforcing that this pattern scales to genuinely complex, multi-faceted coordination needs.

---

## Common Interview Trap

**Trap Question:** _"Since `Condition.signal()` only wakes one thread (like `notify()`), and `signalAll()` wakes all of them (like `notifyAll()`), is there really any difference between using two separate `Condition`s versus just always using `signalAll()` on a single shared wait-set?"_

The trap is assuming that using `signalAll()` everywhere makes the distinction between multiple conditions irrelevant. The correct answer: there's a **significant** difference in **efficiency** and **correctness clarity**. With a single shared wait-set and `notifyAll()`/`signalAll()`, **every** waiting thread (regardless of what it's actually waiting for) wakes up, re-acquires the lock one at a time, re-checks its own condition, and — if not relevant — goes back to waiting, wasting CPU cycles and lock-acquisition overhead on every single irrelevant wakeup. With separate `Condition` objects, you can call `notFull.signal()` and know that **only** producer threads (genuinely waiting for "not full") are woken — consumer threads waiting on `notEmpty` are completely undisturbed. This targeted signaling is precisely the efficiency and clarity advantage that justifies `Condition`'s existence over a single, undifferentiated intrinsic wait-set.

---

## Frequently Asked Questions

**Q: Can `Condition` be used with `synchronized` blocks instead of `ReentrantLock`?** A: No — `Condition` objects are only obtainable via a `Lock` implementation's `newCondition()` method; they have no direct connection to intrinsic `synchronized` blocks.

**Q: Does `signal()` guarantee the awakened thread will run immediately?** A: No — it only makes the thread eligible to compete for re-acquiring the lock; actual scheduling still depends on the OS scheduler (see [[11 - Thread Scheduler]]), exactly like intrinsic `notify()`.

**Q: Is there a limit to how many `Condition` objects a single `Lock` can have?** A: No practical limit — you can create as many as your coordination design genuinely requires.

**Q: Does `ReadWriteLock`/`StampedLock` also support `Condition` objects?** A: `ReentrantReadWriteLock` does (both its read and write locks can create conditions, though write-lock conditions are more commonly used in practice); `StampedLock` does not support `Condition` objects at all, one of its notable limitations (see [[34 - StampedLock]]).

---

## Revision Checklist

- [ ] I understand that `Condition` is `ReentrantLock`'s equivalent of intrinsic `wait()`/`notify()`.
- [ ] I understand the key advantage: multiple, independent wait-sets per lock.
- [ ] I know the core methods (`await()`, `signal()`, `signalAll()`) and their intrinsic equivalents.
- [ ] I understand why the wait condition must always be checked in a `while` loop.
- [ ] I can implement a bounded-buffer-style producer-consumer pattern using two separate `Condition`s.

---

## Flashcards

**Q:** How do you create a `Condition` object? **A:** Via `lock.newCondition()`, where `lock` is a `ReentrantLock` (or other `Lock` implementation).

**Q:** What is `Condition`'s main advantage over intrinsic `wait()`/`notify()`? **A:** Support for multiple, independent wait-sets per lock, enabling targeted signaling.

**Q:** What is the `Condition` equivalent of `Object.notify()`? **A:** `condition.signal()`.

**Q:** What is the `Condition` equivalent of `Object.notifyAll()`? **A:** `condition.signalAll()`.

**Q:** Should the wait condition be checked with `if` or `while` after `await()` returns? **A:** `while` — to guard against spurious wakeups and race conditions between multiple waiters.

---

## References

- Oracle Java Tutorials — Lock Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/newlocks.html
- `java.util.concurrent.locks.Condition` API Documentation
- `java.util.concurrent.locks.ReentrantLock` API Documentation (`newCondition`)