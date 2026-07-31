# BlockingQueue

## Objective

By the end of this note, you will understand:

- What `BlockingQueue` is and how it relates to everything built manually in [[39 - Producer Consumer]] through [[42 - notifyAll()]]
- The main implementations (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`, `PriorityBlockingQueue`) and when to use each
- The four families of insertion/removal methods and their different blocking behaviors
- Why production code should almost always prefer `BlockingQueue` over hand-rolled coordination

This note closes out **Section 7: Inter-Thread Communication** — it's the payoff: the JDK's production-ready implementation of everything this section built by hand.

---

## Why should I learn this?

You've now built the Producer-Consumer pattern from scratch, twice — once with `wait()`/`notifyAll()`, once conceptually with `Condition`. `BlockingQueue` is the JDK's own, thoroughly tested, highly optimized implementation of exactly this pattern. Nearly every real-world Java system that needs thread-safe work queues — job queues, event pipelines, thread pool internals — uses a `BlockingQueue` implementation directly, rather than reimplementing wait/notify logic by hand. This is one of the most immediately useful, practical tools in the entire vault.

---

## Beginner Explanation

`BlockingQueue` is a queue (first-in, first-out, by default) with a superpower: its `put()` and `take()` methods **automatically** handle all the waiting logic you built manually in [[39 - Producer Consumer]] — if the queue is full, `put()` **blocks** until there's room; if the queue is empty, `take()` **blocks** until something is available. All the `synchronized`/`wait()`/`notifyAll()` (or `Lock`/`Condition`) machinery is handled internally, completely hidden from you.

Think of it as buying a professionally engineered, safety-tested product instead of building the same thing yourself from raw materials — you get the same functional result, with far less risk of a subtle mistake, and usually better performance too.

---

## Real World Analogy

**Postal Service Analogy**

Imagine you need a mailbox system where letter carriers (producers) can drop off mail and residents (consumers) can pick it up, safely, even if multiple carriers and residents use the same mailbox simultaneously. You _could_ design and build your own custom mailbox mechanism from scratch, carefully engineering the flap, the lock, and the signaling system to prevent letters from getting crushed or lost during simultaneous access. Or, you could simply install a **commercially manufactured, safety-certified mailbox** — designed by engineers who've already solved every edge case, tested extensively, and refined over years of real-world use.

`BlockingQueue` is exactly this "professionally manufactured mailbox" — the JDK engineers have already solved the coordination problem correctly and efficiently; you just need to know how to install and use it.

---

## Technical Explanation

### The `BlockingQueue` Interface

```java
public interface BlockingQueue<E> extends Queue<E> {
    void put(E e) throws InterruptedException;         // blocks if full
    E take() throws InterruptedException;               // blocks if empty

    boolean offer(E e, long timeout, TimeUnit unit) throws InterruptedException; // bounded wait
    E poll(long timeout, TimeUnit unit) throws InterruptedException;             // bounded wait

    boolean offer(E e);   // non-blocking, returns false if full
    E poll();              // non-blocking, returns null if empty
    // ... plus all standard Queue/Collection methods
}
```

### The Four Families of Methods

|Behavior|Insert|Remove|
|---|---|---|
|**Blocks indefinitely**|`put(e)`|`take()`|
|**Blocks with timeout**|`offer(e, timeout, unit)`|`poll(timeout, unit)`|
|**Non-blocking, immediate**|`offer(e)` (returns `false` if full)|`poll()` (returns `null` if empty)|
|**Throws exception**|`add(e)` (throws if full)|`remove()` (throws if empty)|

This directly mirrors the design choices you saw in [[07 - Future]] (blocking `get()` vs. timed `get(timeout, unit)`) and [[32 - ReentrantLock]] (`lock()` vs. `tryLock()` vs. `tryLock(timeout, unit)`) — a consistent pattern across `java.util.concurrent`.

### Main Implementations

**`ArrayBlockingQueue`** — fixed-capacity, backed by an array:

```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10); // capacity MUST be specified
```

**`LinkedBlockingQueue`** — optionally bounded (defaults to effectively unbounded, `Integer.MAX_VALUE`), backed by a linked structure:

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();      // unbounded
BlockingQueue<String> boundedQueue = new LinkedBlockingQueue<>(10); // bounded
```

**`SynchronousQueue`** — a queue with **zero** internal capacity; every `put()` must be matched by a **simultaneous** `take()` from another thread (a direct hand-off, with no buffering at all):

```java
BlockingQueue<String> handoff = new SynchronousQueue<>();
```

**`PriorityBlockingQueue`** — unbounded, orders elements by natural ordering or a supplied `Comparator`, rather than strict FIFO:

```java
BlockingQueue<Task> priorityQueue = new PriorityBlockingQueue<>(11, Comparator.comparingInt(Task::getPriority));
```

### Full Producer-Consumer Example Using `BlockingQueue`

```java
BlockingQueue<Integer> buffer = new ArrayBlockingQueue<>(2);

// Producer thread
buffer.put(item); // blocks automatically if full -- NO manual wait()/notifyAll() needed!

// Consumer thread
Integer item = buffer.take(); // blocks automatically if empty
```

Compare this to the ~20 lines of `synchronized`/`while`/`wait()`/`notifyAll()` code required in [[39 - Producer Consumer]] — `BlockingQueue` reduces the entire pattern to two method calls, with all the coordination correctness (and typically better performance) handled internally.

---

## Internal Working

`BlockingQueue` implementations are built using the exact same primitives covered throughout this vault, just already correctly assembled:

- `ArrayBlockingQueue` internally uses a `ReentrantLock` with **two** `Condition` objects (`notFull`, `notEmpty`) — literally the exact pattern built by hand in [[35 - Condition]], just professionally implemented and tested.
- `LinkedBlockingQueue` uses **two separate locks** (one for the head/take side, one for the tail/put side) for better concurrency, since put and take operations on a linked structure can, with careful design, avoid contending for the exact same lock in many cases.
- `SynchronousQueue` uses more specialized internal coordination (historically evolving between different internal algorithms across JDK versions) to achieve its direct hand-off semantics with no buffering at all.

Understanding this connects directly back to everything you learned in Sections 4-7: `BlockingQueue` isn't magic — it's the same tools you now understand, expertly assembled.

---

## Memory Diagram

```
        ArrayBlockingQueue(capacity=3)
   -----------------------------------
   |  internal array: [A, B, _]        |
   |  ReentrantLock                    |
   |  Condition notFull                |
   |  Condition notEmpty               |
   -----------------------------------

   put(C) -> array becomes [A, B, C], notEmpty.signal()
   take() -> removes A, array becomes [B, C, _], notFull.signal()

   (Exactly the mechanics from [[35 - Condition]], hidden behind a simple API)
```

---

## Execution Flow

```
Producer calls queue.put(item)
        |
        v
Internally: acquire lock, while(full) await on notFull,
add item, signal notEmpty, release lock
        |
        v
(All handled INSIDE put() -- caller just sees a blocking method call)

Consumer calls queue.take()
        |
        v
Internally: acquire lock, while(empty) await on notEmpty,
remove item, signal notFull, release lock
        |
        v
(All handled INSIDE take() -- caller just sees a blocking method call)
```

---

## Code Example

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;

public class BlockingQueueDemo {

    public static void main(String[] args) throws InterruptedException {

        BlockingQueue<Integer> buffer = new ArrayBlockingQueue<>(2);

        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    System.out.println("Producing: " + i);
                    buffer.put(i); // blocks automatically if buffer is full
                    System.out.println("Produced: " + i + " (size=" + buffer.size() + ")");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Producer");

        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 5; i++) {
                    Thread.sleep(200); // slower consumer
                    Integer item = buffer.take(); // blocks automatically if buffer is empty
                    System.out.println("Consumed: " + item + " (size=" + buffer.size() + ")");
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }, "Consumer");

        producer.start();
        consumer.start();
        producer.join();
        consumer.join();

        System.out.println("--- Demonstrating timed offer/poll ---");
        BlockingQueue<String> smallQueue = new ArrayBlockingQueue<>(1);
        smallQueue.put("only slot");

        boolean added = smallQueue.offer("won't fit", 1, TimeUnit.SECONDS);
        System.out.println("Timed offer succeeded? " + added + " (should be false -- queue stayed full)");

        String polled = smallQueue.poll(1, TimeUnit.SECONDS);
        System.out.println("Timed poll got: " + polled);
    }
}
```

### Line-by-line explanation

1. The producer/consumer logic is **identical in behavior** to [[39 - Producer Consumer]]'s hand-rolled version, but requires **zero** manual synchronization code — `put()`/`take()` handle everything internally.
2. The timed `offer()` demonstrates the bounded-wait family: since `smallQueue` has capacity 1 and is already full, `offer("won't fit", 1, TimeUnit.SECONDS)` waits up to 1 second for room, then returns `false` since none becomes available.
3. `poll(1, TimeUnit.SECONDS)` successfully retrieves the one item, since it's immediately available.

---

## Dry Run

```
Producing: 1
Produced: 1 (size=1)
Producing: 2
Produced: 2 (size=2)
Producing: 3
[... producer blocks here, buffer full ...]
Consumed: 1 (size=1)
Produced: 3 (size=2)
Producing: 4
[... continues alternating ...]
Consumed: 5 (size=0)
--- Demonstrating timed offer/poll ---
Timed offer succeeded? false (should be false -- queue stayed full)
Timed poll got: only slot
```

---

## Common Mistakes

1. **Hand-rolling `wait()`/`notifyAll()`-based coordination in production code instead of using `BlockingQueue`.**
    
    - Unless you have a genuinely custom coordination need beyond what any `BlockingQueue` implementation provides, this is unnecessary risk and effort — always check whether `BlockingQueue` already solves your problem first.
2. **Choosing `LinkedBlockingQueue`'s default (unbounded) constructor without considering memory risk.**
    
    - An unbounded queue removes the natural backpressure mechanism (see [[39 - Producer Consumer]]) — if producers can outpace consumers indefinitely, this can lead to unbounded memory growth. Always consider whether a bounded capacity is more appropriate.
3. **Using `add()`/`remove()` (the exception-throwing family) in code that should instead use `offer()`/`poll()` or `put()`/`take()`.**
    
    - `add()` throwing an unchecked `IllegalStateException` when the queue is full is rarely the desired behavior in a genuine producer-consumer scenario — it's better suited to specific edge cases where a full queue truly represents an unexpected error condition.
4. **Forgetting that `SynchronousQueue` has zero capacity.**
    
    - Calling `put()` on a `SynchronousQueue` with no consumer currently calling `take()` will block indefinitely (or until a consumer appears) — this is by design (a direct hand-off queue), but surprises developers expecting even minimal buffering.

---

## Production Usage

- **`ThreadPoolExecutor`** (the class underlying most `ExecutorService` implementations, see [[44 - Thread Pool]]) uses a `BlockingQueue` internally to hold pending tasks — this is one of the most important real-world uses of `BlockingQueue` in the entire JDK.
- **Log aggregation pipelines**: application threads `put()` log entries into a `BlockingQueue`; a dedicated background thread `take()`s them and writes to disk/network, decoupling application performance from I/O latency.
- **Work distribution systems**: a pool of worker threads all calling `take()` on a shared `BlockingQueue`, naturally load-balancing work as each becomes available (this is essentially how thread pools distribute tasks internally).
- **`SynchronousQueue`** is specifically used internally by `Executors.newCachedThreadPool()` for direct task hand-off to newly-created or idle threads, without any buffering delay.

---

## Interview Questions

**Beginner:**

1. What is `BlockingQueue`, and what problem does it solve?
2. What does `put()` do if the queue is full?
3. What does `take()` do if the queue is empty?

**Intermediate:** 4. What are the four families of insertion/removal methods, and how do they differ? 5. What's the difference between `ArrayBlockingQueue` and `LinkedBlockingQueue`? 6. What is `SynchronousQueue`, and how does its "zero capacity" behavior differ from a normal bounded queue?

**Advanced:** 7. How does `ArrayBlockingQueue` implement its blocking behavior internally, in terms of the primitives covered earlier in this vault? 8. Why might `LinkedBlockingQueue` offer better concurrency than `ArrayBlockingQueue` under certain conditions?

**Production-Level:** 9. What JDK class, central to thread pools, relies directly on `BlockingQueue` internally? 10. Why is choosing between a bounded and unbounded queue an important production design decision, not just an implementation detail?

---

## Follow-up Questions

- Is `BlockingQueue` thread-safe for multiple producers and multiple consumers simultaneously? _(Yes — this is precisely its core design purpose; all implementations correctly handle any number of concurrent producer and consumer threads.)_
- Can you peek at the next element without removing it? _(Yes — `peek()` returns the head element without removing it, returning `null` if empty; this is a non-blocking, `Queue`-inherited method.)_
- Does `BlockingQueue` guarantee FIFO ordering? _(`ArrayBlockingQueue` and `LinkedBlockingQueue` do, by default; `PriorityBlockingQueue` explicitly does NOT — it orders by priority instead; `SynchronousQueue` has no ordering concept at all, since it never buffers more than one hand-off at a time.)_

---

## Best Practices

- Default to `BlockingQueue` (choosing the appropriate implementation) over hand-rolled `wait()`/`notify()` or `Lock`/`Condition` coordination for any standard producer-consumer need.
- Prefer bounded queues (`ArrayBlockingQueue`, or a bounded `LinkedBlockingQueue`) in most production scenarios, to provide natural backpressure and avoid unbounded memory growth.
- Choose `put()`/`take()` for standard blocking behavior, timed `offer()`/`poll()` when you need bounded waiting, and the exception-throwing `add()`/`remove()` only for genuinely exceptional "this should never actually be full/empty" scenarios.
- Use `PriorityBlockingQueue` when task ordering matters beyond simple arrival order; use `SynchronousQueue` specifically for direct, unbuffered hand-off scenarios.

---

## Performance Notes

- `BlockingQueue` implementations are generally highly optimized — often outperforming naive hand-rolled equivalents, thanks to careful internal engineering (e.g., `LinkedBlockingQueue`'s dual-lock design for put/take concurrency).
- Bounded queue capacity choice affects both memory usage and how much "buffering slack" exists between producer and consumer speed differences — this is a genuine tuning parameter worth considering based on your specific workload's burstiness.
- `SynchronousQueue`'s zero-buffering design can maximize responsiveness (immediate hand-off) at the cost of tightly coupling producer and consumer timing — appropriate for specific low-latency hand-off scenarios, not general-purpose buffering.

---

## Comparison Table

|Implementation|Capacity|Ordering|Best For|
|---|---|---|---|
|`ArrayBlockingQueue`|Fixed (required)|FIFO|Predictable, bounded buffering|
|`LinkedBlockingQueue`|Optional bound (default unbounded)|FIFO|General-purpose, potentially high-throughput queuing|
|`SynchronousQueue`|Zero (direct hand-off)|N/A|Low-latency, tightly-coupled hand-off|
|`PriorityBlockingQueue`|Unbounded|Priority-based|Task scheduling by importance, not arrival order|

---

## Summary

- `BlockingQueue` is the JDK's production-ready implementation of the Producer-Consumer pattern, internally using the exact `Lock`/`Condition` (or similar) mechanisms covered throughout Sections 4-7.
- `put()`/`take()` block automatically; timed `offer()`/`poll()` provide bounded waiting; non-blocking `offer()`/`poll()` and exception-throwing `add()`/`remove()` round out the full method family.
- Main implementations: `ArrayBlockingQueue` (fixed capacity), `LinkedBlockingQueue` (optionally bounded), `SynchronousQueue` (zero-capacity direct hand-off), `PriorityBlockingQueue` (priority-ordered).
- `BlockingQueue` underlies `ThreadPoolExecutor`'s internal task queue, making it foundational to how Java's own thread pools work.
- Production code should almost always prefer `BlockingQueue` over hand-rolled coordination, reserving manual `wait()`/`notify()` or `Condition`-based implementation for genuinely custom needs.

---

## Cheat Sheet

|Method Family|Insert|Remove|Behavior|
|---|---|---|---|
|Blocking|`put(e)`|`take()`|Waits indefinitely|
|Timed|`offer(e, t, unit)`|`poll(t, unit)`|Waits up to timeout|
|Non-blocking|`offer(e)`|`poll()`|Returns immediately (false/null)|
|Throwing|`add(e)`|`remove()`|Throws exception|

---

## Keywords

BlockingQueue, ArrayBlockingQueue, LinkedBlockingQueue, SynchronousQueue, PriorityBlockingQueue, put, take, ThreadPoolExecutor

---

## Related Notes

[[39 - Producer Consumer]] [[35 - Condition]] [[44 - Thread Pool]] [[46 - ExecutorService]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's producer-consumer demo using `ArrayBlockingQueue`, confirming correct blocking behavior.
2. Compare the code length/complexity to your hand-rolled version from [[39 - Producer Consumer]]'s exercises.

**Medium:** 3. Experiment with `SynchronousQueue`: create a producer that calls `put()` with no consumer running yet, confirm it blocks, then start a consumer and confirm the hand-off occurs immediately. 4. Implement a `PriorityBlockingQueue`-based task scheduler where tasks with lower priority numbers are processed first, and confirm the processing order matches priority, not insertion order.

**Hard:** 5. Build a small benchmark comparing throughput of a hand-rolled `wait()`/`notifyAll()` bounded buffer against `ArrayBlockingQueue` for an equivalent workload, and analyze the performance difference.

---

## Mini Project

**Project: Multi-Stage Pipeline (BlockingQueue Edition)**

Build a small console application that:

1. Reimplements the multi-stage pipeline mini project from [[39 - Producer Consumer]] ("Fetch" → buffer → "Transform" → buffer → "Save"), this time using `ArrayBlockingQueue` for both buffers instead of hand-rolled coordination.
2. Runs multiple threads at each stage, confirms correct end-to-end flow, and prints throughput statistics.
3. Compares the total lines of code and complexity against the original hand-rolled version.

**Goal:** Directly experience the practical payoff of everything learned in this section — building the same real system with dramatically less code and risk, using the JDK's professionally engineered tools.

---

## Common Interview Trap

**Trap Question:** _"Since `BlockingQueue` already handles all the synchronization internally, is it ever necessary to add your own additional `synchronized` blocks when working with it?"_

The trap is assuming `BlockingQueue`'s internal thread-safety automatically makes **any** code that uses it fully thread-safe. The correct, more nuanced answer: `BlockingQueue`'s own operations (`put()`, `take()`, `size()`, etc.) are individually thread-safe, but — exactly like the `ConcurrentHashMap` lesson from [[26 - Race Condition]] — a **sequence** of multiple operations on it (or coordination involving the queue **plus** other shared state) is not automatically atomic as a combined unit. For example, "check `queue.size()`, then decide whether to `put()` based on some external condition" can still be a check-then-act race if that combined logic isn't itself properly synchronized. `BlockingQueue` solves the specific, well-defined problem of safe, blocking insertion/removal — it does not automatically make all surrounding application logic that uses it thread-safe.

---

## Frequently Asked Questions

**Q: Can `BlockingQueue` hold `null` elements?** A: No — all standard JDK `BlockingQueue` implementations explicitly disallow `null` elements, throwing `NullPointerException` if attempted; this is a deliberate design choice, since `null` is used as a sentinel return value (e.g., by non-blocking `poll()`) to indicate "empty."

**Q: Is `BlockingQueue` part of the `java.util.concurrent` package?** A: Yes — along with all its major implementations (`ArrayBlockingQueue`, `LinkedBlockingQueue`, etc.).

**Q: Does `BlockingQueue` support iteration?** A: Yes — it extends `Collection`/`Queue`, supporting iteration, though be aware that a `BlockingQueue`'s iterator may have weaker consistency guarantees under concurrent modification compared to a purely single-threaded collection; the primary intended usage pattern remains `put()`/`take()`, not general iteration.

**Q: Does `ThreadPoolExecutor` let you choose which `BlockingQueue` implementation it uses internally?** A: Yes — this is actually a significant, commonly-tuned configuration choice (bounded `ArrayBlockingQueue` vs. unbounded `LinkedBlockingQueue` vs. `SynchronousQueue`) that meaningfully affects a thread pool's behavior under load, covered further in [[44 - Thread Pool]] and [[46 - ExecutorService]].

---

## Revision Checklist

- [ ] I understand what `BlockingQueue` provides and how it relates to the manually-built Producer-Consumer pattern.
- [ ] I know the four families of insertion/removal methods and their blocking behaviors.
- [ ] I can choose the appropriate implementation (`ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`, `PriorityBlockingQueue`) for a given scenario.
- [ ] I understand that `BlockingQueue`'s individual operations are thread-safe, but sequences of operations combined with other logic are not automatically atomic.
- [ ] I know that `ThreadPoolExecutor` uses `BlockingQueue` internally for its task queue.

---

## Flashcards

**Q:** What does `put()` do if a `BlockingQueue` is full? **A:** Blocks (waits) until space becomes available.

**Q:** What does `take()` do if a `BlockingQueue` is empty? **A:** Blocks (waits) until an item becomes available.

**Q:** What is `SynchronousQueue`'s capacity? **A:** Zero — every `put()` must be matched by a simultaneous `take()`.

**Q:** Which `BlockingQueue` implementation orders elements by priority rather than FIFO? **A:** `PriorityBlockingQueue`.

**Q:** What central JDK class uses `BlockingQueue` internally for its task queue? **A:** `ThreadPoolExecutor`.

---

## References

- Oracle Java Tutorials — Concurrent Collections: https://docs.oracle.com/javase/tutorial/essential/concurrency/collections.html
- `java.util.concurrent.BlockingQueue` API Documentation
- `java.util.concurrent.ArrayBlockingQueue` / `LinkedBlockingQueue` / `SynchronousQueue` / `PriorityBlockingQueue` API Documentation