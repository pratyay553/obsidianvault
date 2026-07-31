# Producer Consumer

## Objective

By the end of this note, you will understand:

- The Producer-Consumer pattern as a foundational concurrency design pattern, not just an academic exercise
- How to implement it correctly using intrinsic `wait()`/`notify()`
- How to implement it using `ReentrantLock` + `Condition` (from [[35 - Condition]])
- Why this pattern underlies almost every real-world queue-based system you'll ever build

This note closes out **Section 6: Concurrency Problems**, shifting from "problems to avoid" (deadlock, livelock, starvation) to "the classic pattern that, done right, avoids all of them while solving a genuinely common real need."

---

## Why should I learn this?

The Producer-Consumer pattern is not an obscure academic puzzle — it's the **conceptual foundation** of message queues, job queues, logging pipelines, event processing systems, and virtually every "some threads generate work, other threads process it" architecture in real software. Understanding it deeply — including the subtle bugs beginners introduce — prepares you directly for [[40 - wait()]], [[41 - notify()]], [[42 - notifyAll()]], and [[43 - BlockingQueue]] (which is the production-ready, pre-built solution to this exact problem).

---

## Beginner Explanation

Imagine a bakery: bakers (**producers**) continuously make fresh bread and place it on a shared shelf. Customers (**consumers**) continuously take bread from that same shelf. Two rules keep this working smoothly:

1. If the shelf is **full**, bakers must **pause** — there's no room for more bread until customers take some away.
2. If the shelf is **empty**, customers must **pause** — there's nothing to take until bakers make more.

This is the entire Producer-Consumer pattern: one or more threads **producing** items into a shared, bounded buffer, and one or more threads **consuming** items from it, with proper coordination so producers don't overflow the buffer and consumers don't try to take from an empty one.

---

## Real World Analogy

**Restaurant Kitchen Analogy**

Imagine a restaurant's kitchen pass — the shelf where chefs (producers) place finished dishes, and waiters (consumers) pick them up to deliver to tables. The pass has limited space:

- If the pass is full of completed dishes waiting for pickup, chefs must pause plating new dishes until waiters clear some away.
- If the pass is empty, waiters simply wait for the next dish to appear rather than repeatedly checking an empty shelf in a tight, wasteful loop.

A well-run kitchen coordinates this smoothly — chefs and waiters aren't constantly bumping into each other checking the pass; there's an implicit (or explicit, via a bell) signaling system: "dish ready!" and "pass has room now!"

---

## Technical Explanation

### Implementation 1: Using Intrinsic `wait()`/`notify()`

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }

    public synchronized void put(T item) throws InterruptedException {
        while (queue.size() == capacity) {
            wait(); // release the monitor, wait to be notified
        }
        queue.add(item);
        notifyAll(); // wake up any waiting consumers (and other producers)
    }

    public synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        T item = queue.poll();
        notifyAll(); // wake up any waiting producers (and other consumers)
        return item;
    }
}
```

**Key design decisions**:

- `while`, not `if`, guards the wait condition — protecting against spurious wakeups and against another thread "stealing" the resource between notification and re-acquisition (see [[40 - wait()]]).
- `notifyAll()`, not `notify()`, is used here because both producers and consumers share the **same** single intrinsic wait-set — using plain `notify()` risks waking the "wrong kind" of waiting thread (e.g., waking another producer when a consumer actually needed to be woken), which could leave the system stalled. This exact limitation is what [[35 - Condition]] was designed to solve.

### Implementation 2: Using `ReentrantLock` + Two `Condition`s (Preferred, More Efficient)

This is precisely the `BoundedBuffer` example already built out in full in [[35 - Condition]] — using separate `notFull` and `notEmpty` conditions allows **targeted** signaling (only wake the specific kind of thread that can actually proceed), rather than `notifyAll()`'s "wake everyone and let them sort it out" approach.

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();

    // put() and take() as shown in [[35 - Condition]]
}
```

### Why the Pattern Prevents (Correctly Implemented) the Problems from This Section

- **No deadlock**: there's only ever one lock/monitor involved per buffer, and both producers and consumers acquire it in the same, single, consistent way — no circular wait is possible.
- **No livelock**: `wait()`/`await()` properly suspends the thread (not a busy-spin loop), so there's no "constantly retrying and reacting" CPU-burning behavior — a correctly blocked/waiting thread consumes zero CPU until properly signaled.
- **No (severe) starvation**: as long as `notifyAll()`/`signalAll()` (or well-designed `signal()` targeting) is used correctly, all waiting threads get a fair chance to recheck their condition each time a relevant change occurs.

---

## Internal Working

Step-by-step of a producer calling `put()` when the buffer is full:

1. Producer acquires the monitor/lock.
2. Checks `queue.size() == capacity` — true, so calls `wait()`/`await()`.
3. This **releases** the lock and suspends the producer (state: `WAITING`).
4. A consumer acquires the lock, calls `take()`, removes an item, and calls `notifyAll()`/`notFull.signal()`.
5. The producer wakes up, **re-acquires** the lock (this may itself involve a brief wait if other threads are also competing), and **re-checks** its `while` condition — since an item was just removed, the buffer is no longer full, so it proceeds to add its item.

---

## Memory Diagram

```
        Shared Bounded Buffer (capacity 3)
   -----------------------------------
   |  [item1, item2, item3]  (FULL)   |
   -----------------------------------
        ^                      |
        |                      v
   Producer: put()        Consumer: take()
   sees FULL, calls        removes item1,
   wait() -- releases      calls notifyAll()
   lock, suspends                |
        ^                        |
        |________________________|
        Producer wakes, re-acquires lock,
        re-checks: no longer full, adds item
```

---

## Execution Flow

```
Producer: acquire lock -> check full? -> YES -> wait() (release lock, suspend)
        |
   (meanwhile)
        |
Consumer: acquire lock -> check empty? -> NO -> remove item -> notifyAll() -> release lock
        |
        v
Producer wakes -> re-acquires lock -> re-check full? -> NO -> add item -> notifyAll() -> release lock
```

---

## Code Example

```java
import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumerDemo {

    static class BoundedBuffer<T> {
        private final Queue<T> queue = new LinkedList<>();
        private final int capacity;

        BoundedBuffer(int capacity) {
            this.capacity = capacity;
        }

        public synchronized void put(T item) throws InterruptedException {
            while (queue.size() == capacity) {
                System.out.println(Thread.currentThread().getName() + " waiting: buffer FULL.");
                wait();
            }
            queue.add(item);
            System.out.println(Thread.currentThread().getName() + " produced: " + item + " (size=" + queue.size() + ")");
            notifyAll();
        }

        public synchronized T take() throws InterruptedException {
            while (queue.isEmpty()) {
                System.out.println(Thread.currentThread().getName() + " waiting: buffer EMPTY.");
                wait();
            }
            T item = queue.poll();
            System.out.println(Thread.currentThread().getName() + " consumed: " + item + " (size=" + queue.size() + ")");
            notifyAll();
            return item;
        }
    }

    public static void main(String[] args) throws InterruptedException {
        BoundedBuffer<Integer> buffer = new BoundedBuffer<>(2);

        Runnable producer = () -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    buffer.put(i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Runnable consumer = () -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    Thread.sleep(200); // consume slower than production
                    buffer.take();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        };

        Thread producerThread = new Thread(producer, "Producer");
        Thread consumerThread = new Thread(consumer, "Consumer");

        producerThread.start();
        consumerThread.start();

        producerThread.join();
        consumerThread.join();

        System.out.println("Both threads finished. Final buffer size: " + buffer.queue.size());
    }
}
```

### Line-by-line explanation

1. `BoundedBuffer` with capacity 2 uses classic intrinsic `wait()`/`notifyAll()`.
2. The producer, being faster, will quickly fill the buffer to capacity and then print "waiting: buffer FULL," blocking until the slower consumer creates room.
3. The consumer, being slower (200ms delay per item), will occasionally print "waiting: buffer EMPTY" if it happens to catch up to an empty buffer momentarily.
4. Both correctly coordinate to produce and consume all 5 items with a final buffer size of 0, demonstrating correct, deadlock-free, livelock-free coordination.

---

## Dry Run

Example output (exact interleaving varies):

```
Producer produced: 1 (size=1)
Producer produced: 2 (size=2)
Producer waiting: buffer FULL.
Consumer consumed: 1 (size=1)
Producer produced: 3 (size=2)
Producer waiting: buffer FULL.
Consumer consumed: 2 (size=1)
Producer produced: 4 (size=2)
Producer waiting: buffer FULL.
Consumer consumed: 3 (size=1)
Producer produced: 5 (size=2)
Consumer consumed: 4 (size=1)
Consumer consumed: 5 (size=0)
Both threads finished. Final buffer size: 0
```

---

## Common Mistakes

1. **Using `if` instead of `while` for the wait condition.**
    
    - As emphasized throughout, this is unsafe — after waking, the condition must be re-verified, since another thread might have changed it (or a spurious wakeup might have occurred) before this thread actually resumes.
2. **Using `notify()` instead of `notifyAll()` when both producers and consumers share the same monitor.**
    
    - `notify()` wakes an **arbitrary** single waiting thread — if it happens to wake another producer (when a consumer was actually needed to make progress) or vice versa, the system can end up in a prolonged, inefficient (though not technically deadlocked) stall. `notifyAll()` is the safe default when multiple different "kinds" of waiters share one wait-set.
3. **Forgetting `synchronized` on both `put()` and `take()`, or using different lock objects for each.**
    
    - Both methods must synchronize on the **same** object for the shared queue's mutual exclusion and wait/notify signaling to work correctly.
4. **Reinventing this pattern in production code instead of using `BlockingQueue`.**
    
    - The JDK's `BlockingQueue` implementations (see [[43 - BlockingQueue]]) already correctly and efficiently implement this exact pattern — hand-rolling it (as this note does, for learning purposes) is valuable for understanding, but production code should almost always prefer the battle-tested, pre-built utility.

---

## Production Usage

- **Message queues and job queues** (in-memory or backed by systems like Kafka/RabbitMQ at a larger scale) are, at their conceptual core, producer-consumer systems.
- **Logging pipelines**: application threads (producers) write log entries to a queue; a dedicated background thread (consumer) pulls from the queue and writes to disk/network, decoupling slow I/O from the application's main work.
- **Thread pool task queues** (see [[44 - Thread Pool]], [[45 - Executor Framework]]) are themselves a producer-consumer system: submitting code produces tasks, worker threads consume and execute them.
- **Data processing pipelines**: one stage produces intermediate results consumed by the next stage, often via a `BlockingQueue`, forming a multi-stage producer-consumer chain.

---

## Interview Questions

**Beginner:**

1. What is the Producer-Consumer pattern?
2. Why does a producer need to "wait" when the buffer is full?
3. Why does a consumer need to "wait" when the buffer is empty?

**Intermediate:** 4. Why must the wait condition be checked with `while`, not `if`? 5. Why is `notifyAll()` generally safer than `notify()` in this pattern when using a single shared monitor? 6. What real-world system is a canonical example of the Producer-Consumer pattern at scale?

**Advanced:** 7. How does using separate `Condition` objects (from [[35 - Condition]]) improve on the plain `wait()`/`notifyAll()` implementation of this pattern? 8. Explain how the Producer-Consumer pattern, correctly implemented, avoids deadlock, livelock, and severe starvation.

**Production-Level:** 9. Why would you use `BlockingQueue` instead of hand-rolling this pattern in production code? 10. Describe a real production data pipeline you'd design using the producer-consumer pattern, including where you'd place the bounded buffer and why bounding it matters.

---

## Follow-up Questions

- What happens if the buffer has **no** capacity limit (unbounded)? _(Producers would never need to wait for space, simplifying one side of the coordination — but this introduces a different risk: if producers are faster than consumers indefinitely, the buffer can grow without bound, potentially causing memory exhaustion; bounding the buffer is often a deliberate backpressure mechanism.)_
- Can there be multiple producer AND multiple consumer threads simultaneously? _(Yes — the pattern generalizes naturally to many producers and many consumers sharing the same bounded buffer, which is exactly why `while` loops and `notifyAll()`/proper condition signaling matter even more, since more threads are competing to recheck the shared state.)_
- Is `wait()`/`notifyAll()` the only way to implement this pattern? _(No — as shown, `ReentrantLock`+`Condition` is an alternative, generally offering better signaling precision via separate conditions; and in production, `BlockingQueue` provides a ready-made, optimized implementation requiring no manual wait/notify code at all.)_

---

## Best Practices

- Always use `while` loops to guard wait conditions, never `if`.
- Use `notifyAll()` (or targeted `Condition.signal()` calls, if using separate conditions) rather than a bare `notify()` when multiple different kinds of waiting threads share a wait-set.
- Bound your buffer's capacity deliberately, as a backpressure mechanism preventing unbounded memory growth if producers outpace consumers.
- In production code, prefer `BlockingQueue` (see [[43 - BlockingQueue]]) over hand-rolling this pattern — reserve manual implementation for learning or genuinely custom coordination needs beyond what `BlockingQueue` provides.

---

## Performance Notes

- Intrinsic `wait()`/`notifyAll()` waking **all** waiting threads (even irrelevant ones) on every state change adds some overhead compared to targeted `Condition`-based signaling — a meaningful consideration under high contention with many producer/consumer threads.
- `BlockingQueue` implementations are generally highly optimized (some using more advanced internal techniques than a simple lock+condition pair) and should be preferred for production performance, not just convenience.

---

## Comparison Table

|Implementation|Signaling Precision|Complexity|Recommended For|
|---|---|---|---|
|`synchronized` + `wait()`/`notifyAll()`|Low (wakes everyone)|Low|Learning, simple cases|
|`ReentrantLock` + separate `Condition`s|High (targeted)|Medium|Custom coordination needs|
|`BlockingQueue` (JDK built-in)|High (internally optimized)|Lowest (just use the API)|Production code, standard use cases|

---

## Summary

- The Producer-Consumer pattern coordinates one or more producer threads adding items to a shared, bounded buffer with one or more consumer threads removing items from it.
- Producers wait when the buffer is full; consumers wait when it's empty — implemented via `wait()`/`notifyAll()` (intrinsic) or `Lock`+`Condition` (explicit, more precise).
- Correctly implemented (using `while` loops and proper signaling), this pattern is free of deadlock, livelock, and severe starvation.
- This pattern underlies message queues, job queues, logging pipelines, and thread pools throughout real production systems.
- Production code should prefer the JDK's `BlockingQueue` over hand-rolling this pattern, once you understand the underlying mechanics (which this note and [[35 - Condition]] cover).

---

## Cheat Sheet

|Rule|Why|
|---|---|
|Use `while`, not `if`|Guards against spurious wakeups and stolen state changes|
|Use `notifyAll()` (or targeted `Condition.signal()`)|Avoids waking the "wrong kind" of waiter with plain `notify()`|
|Bound the buffer|Prevents unbounded memory growth; provides backpressure|
|Prefer `BlockingQueue` in production|Pre-built, optimized, battle-tested|

---

## Keywords

Producer-Consumer, Bounded Buffer, wait, notifyAll, Condition, BlockingQueue, Backpressure

---

## Related Notes

[[35 - Condition]] [[40 - wait()]] [[41 - notify()]] [[42 - notifyAll()]] [[43 - BlockingQueue]] [[44 - Thread Pool]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `BoundedBuffer` demo and confirm correct producer/consumer coordination.
2. Change the buffer capacity to 1 and observe the more frequent "waiting" messages from both sides.

**Medium:** 3. Rewrite the demo using `ReentrantLock` + two separate `Condition`s (as in [[35 - Condition]]), and compare the code and behavior. 4. Add a second producer and a second consumer thread, and confirm the pattern still coordinates correctly with multiple threads on each side.

**Hard:** 5. Rewrite the demo using the JDK's `BlockingQueue` (specifically `ArrayBlockingQueue`) instead of a hand-rolled buffer, comparing code simplicity to your manual implementation.

---

## Mini Project

**Project: Multi-Stage Processing Pipeline**

Build a small console application that:

1. Implements a 3-stage pipeline: "Fetch" (producer) → bounded buffer → "Transform" (consumer AND producer for the next stage) → bounded buffer → "Save" (final consumer).
2. Uses the hand-rolled `wait()`/`notifyAll()` bounded buffer pattern from this note for both buffers.
3. Runs multiple threads at each stage, confirms items flow correctly end-to-end, and prints throughput statistics for each stage.
4. As a bonus, reimplement the entire pipeline using `BlockingQueue` instead, and compare code complexity.

**Goal:** Build a realistic, multi-stage producer-consumer system — exactly the shape of many real data-processing pipelines — reinforcing both the manual mechanics and the value of the JDK's built-in alternative.

---

## Common Interview Trap

**Trap Question:** _"If both producers and consumers use `notify()` (not `notifyAll()`) after modifying the shared buffer, is this still correct, just less efficient?"_

The trap is assuming `notify()` is merely a "less efficient but still correct" choice. The correct, more precise answer: it can actually be **incorrect**, not just less efficient. Since `notify()` wakes an **arbitrary** thread from the single shared wait-set — which may contain **both** waiting producers and waiting consumers — it's entirely possible for a producer's `notify()` (intended to wake a waiting consumer, since an item was just added) to instead wake **another waiting producer**, which will simply re-check its own "is it full?" condition, find it still isn't relevant to it, and go back to waiting — potentially leaving the actual intended consumer, and the item now available for it, permanently unnoticed if no further "lucky" notify happens to reach it. This isn't merely suboptimal — it can cause a **genuine, indefinite stall** in some unlucky interleavings. `notifyAll()` (or separate, targeted `Condition`s) is the only fully safe choice when multiple different kinds of waiting threads share one signaling mechanism.

---

## Frequently Asked Questions

**Q: Does the Producer-Consumer pattern require exactly one producer and one consumer?** A: No — it generalizes naturally to multiple producers and/or multiple consumers sharing the same buffer, as long as the synchronization (locking, wait/notify or condition signaling) is implemented correctly for the multi-threaded case on both sides.

**Q: Is an unbounded buffer ever appropriate?** A: Sometimes, if memory constraints genuinely aren't a concern and you specifically want producers to never block — but in most real systems, a bounded buffer providing backpressure is the safer, more common default design choice.

**Q: How does `BlockingQueue` relate to everything in this note?** A: `BlockingQueue` (see [[43 - BlockingQueue]]) is the JDK's pre-built, production-ready implementation of exactly this pattern — its `put()` and `take()` methods behave conceptually identically to this note's hand-rolled versions, but with a well-tested, often more efficient internal implementation.

**Q: Is the Producer-Consumer pattern specific to Java, or a general software design pattern?** A: It's a completely general, language-agnostic concurrent programming pattern — used in virtually every programming language and system that deals with asynchronous work distribution; Java's specific tools (`wait()`/`notify()`, `Condition`, `BlockingQueue`) are just Java's implementations of this universal concept.

---

## Revision Checklist

- [ ] I can explain the Producer-Consumer pattern and why bounding the buffer matters.
- [ ] I can implement it correctly using `synchronized` + `wait()`/`notifyAll()`.
- [ ] I understand why `notifyAll()` is safer than `notify()` in a shared-monitor implementation.
- [ ] I understand how `Condition`-based signaling improves on the plain intrinsic version.
- [ ] I know that production code should generally prefer `BlockingQueue` over a hand-rolled implementation.

---

## Flashcards

**Q:** What does a producer do when the shared buffer is full? **A:** Waits (releasing the lock) until a consumer removes an item and signals.

**Q:** What does a consumer do when the shared buffer is empty? **A:** Waits until a producer adds an item and signals.

**Q:** Why is `notifyAll()` generally preferred over `notify()` in this pattern? **A:** Because `notify()` might wake the "wrong kind" of waiting thread, potentially causing an indefinite stall.

**Q:** What JDK utility implements this pattern in a production-ready way? **A:** `BlockingQueue`.

**Q:** Why bound the buffer's capacity rather than leaving it unbounded? **A:** To provide backpressure and prevent unbounded memory growth if producers outpace consumers.

---

## References

- Oracle Java Tutorials — Guarded Blocks: https://docs.oracle.com/javase/tutorial/essential/concurrency/guardmeth.html
- `java.util.concurrent.BlockingQueue` API Documentation
- `java.util.concurrent.locks.Condition` API Documentation