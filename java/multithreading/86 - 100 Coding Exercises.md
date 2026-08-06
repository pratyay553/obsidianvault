# 86 - 100 Coding Exercises

Tags: #practice #section15
Related: [[87 - 50 Debugging Exercises]] | [[88 - 25 Mini Projects]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
100 hands-on coding exercises covering every concept in the vault — from beginner to production-level.

---

## Section 1 — Thread Basics (1–10)

1. Create a thread using `extends Thread` that prints numbers 1–10 with its thread name.
2. Create the same thread using `implements Runnable` and a lambda.
3. Start 5 threads simultaneously and print their names. Observe the output ordering.
4. Create a thread, call `start()`, then try calling `start()` again. Catch and print the exception.
5. Print the thread ID, name, priority, and daemon status of the main thread.
6. Create a daemon thread that prints "heartbeat" every 200ms. Observe it stop when main exits.
7. Set thread priorities 1, 5, and 10 on three threads counting to 10M. Compare counts.
8. Name threads sequentially: `worker-0`, `worker-1`, … using `AtomicInteger`.
9. Create a thread that sets its own name inside `run()` based on a processed item.
10. Use `Thread.getAllStackTraces()` to print all currently running JVM thread names.

---

## Section 2 — Thread Lifecycle (11–20)

11. Print a thread's state at each transition: NEW → RUNNABLE → TIMED_WAITING → TERMINATED.
12. Make Thread A wait for Thread B using `join()`. Print messages before and after join.
13. Make Thread A wait for Thread B with a 500ms timeout using `join(500)`. Show both paths.
14. Use `Thread.sleep(1000)` inside a loop that checks `Thread.currentThread().isInterrupted()`.
15. Interrupt a sleeping thread and catch `InterruptedException`. Restore the interrupt flag.
16. Create a thread in WAITING state using `Object.wait()`. Verify with `getState()`.
17. Create a thread in BLOCKED state by holding a lock on another thread. Verify with `getState()`.
18. Write a method that returns only after a condition is true, using `wait()`/`notifyAll()`.
19. Implement a simple start-gun pattern: 5 threads wait on a latch, then all start simultaneously.
20. Implement a completion gate: main waits for 3 worker threads to finish.

---

## Section 3 — Synchronization (21–35)

21. Implement a non-thread-safe counter and demonstrate the bug with 10 threads × 10,000 increments.
22. Fix the counter using `synchronized` method. Verify the count is exact.
23. Fix the counter using `synchronized` block on a specific lock object.
24. Fix the counter using `AtomicInteger`. Compare performance with `synchronized`.
25. Fix the counter using `LongAdder`. Compare performance with `AtomicInteger`.
26. Implement a thread-safe singleton using double-checked locking with `volatile`.
27. Implement a thread-safe singleton using initialization-on-demand holder pattern.
28. Write a `synchronized` method that calls another `synchronized` method on the same object. Verify reentrancy.
29. Use `volatile` to stop a spinning background thread from main.
30. Implement a simple spin-lock using `AtomicBoolean.compareAndSet()`.
31. Demonstrate the ABA problem with an `AtomicInteger`. Then fix with `AtomicStampedReference`.
32. Implement a thread-safe stack using `AtomicReference` and CAS.
33. Use `synchronized` to protect a `HashMap`. Then replace with `ConcurrentHashMap` and compare.
34. Use `Collections.synchronizedList` and demonstrate that compound `get`+`add` still needs external sync.
35. Implement a counter that uses `StripedLock` (simulate with array of locks).

---

## Section 4 — Advanced Locks (36–45)

36. Implement a bounded buffer using `ReentrantLock` + two `Condition`s (not-full, not-empty).
37. Use `ReentrantLock.tryLock(500, ms)` to avoid blocking indefinitely.
38. Use `lockInterruptibly()` and interrupt a waiting thread from outside.
39. Implement a read/write cache using `ReadWriteLock` — multiple concurrent readers, exclusive writer.
40. Use `StampedLock` optimistic read to read a point (x, y) — retry if write occurred during read.
41. Implement a fair `ReentrantLock` and show that threads are served in FIFO order.
42. Use a `Condition` to implement a blocking queue without using `BlockingQueue`.
43. Implement `await(condition)` and `signal()` using `Condition.await()`/`signal()`.
44. Compare `synchronized + wait/notify` vs `ReentrantLock + Condition` for the same use case.
45. Implement `tryLock` with retry and exponential backoff.

---

## Section 5 — Classic Problems (46–55)

46. Implement Producer-Consumer with `BlockingQueue<Integer>`.
47. Implement Producer-Consumer with `wait()`/`notifyAll()` and a bounded buffer.
48. Implement the Dining Philosophers problem without deadlock.
49. Implement the Readers-Writers problem with writer priority using `ReadWriteLock`.
50. Implement a simple deadlock with two threads and two locks. Verify with `jstack`.
51. Fix the deadlock by enforcing lock ordering.
52. Implement a livelock scenario and fix it with random backoff.
53. Demonstrate thread starvation with many high-priority threads and one low-priority thread.
54. Implement the Sleeping Barber problem.
55. Implement a print-spooler: only one thread prints at a time using `Semaphore(1)`.

---

## Section 6 — Executors & Futures (56–65)

56. Submit 10 `Callable` tasks returning random integers. Collect results with `invokeAll`.
57. Submit 5 tasks with `invokeAny` — return the first to complete.
58. Implement a fixed thread pool that processes a list of URLs (simulated). Measure total time.
59. Cancel a running `Future` using `cancel(true)`. Verify the thread responds to interrupt.
60. Submit a task that throws an exception. Catch it from `future.get()`.
61. Use `ScheduledExecutorService` to print timestamps every second for 10 seconds.
62. Use `scheduleAtFixedRate` vs `scheduleWithFixedDelay` — observe the timing difference.
63. Implement a `ForkJoinPool` task to compute Fibonacci(40) using `RecursiveTask`.
64. Use `parallelStream()` to process a list of 1M integers. Compare to sequential stream.
65. Build a `CompletableFuture` pipeline: fetch user → fetch orders → merge → save.

---

## Section 7 — CompletableFuture (66–72)

66. Chain `thenApply`, `thenAccept`, `thenRun` on a `CompletableFuture`.
67. Use `thenCompose` to chain two async operations (one depends on the other's result).
68. Use `thenCombine` to merge results from two independent futures.
69. Use `allOf` to wait for 5 futures and collect all results.
70. Use `anyOf` to return the fastest of 3 futures.
71. Handle exceptions with `exceptionally` and `handle`. Show the difference.
72. Use `CompletableFuture.supplyAsync(supplier, customExecutor)` on a named thread pool.

---

## Section 8 — Concurrent Collections & Coordination (73–82)

73. Use `ConcurrentHashMap.computeIfAbsent` to build a word-frequency map from 4 threads.
74. Use `ConcurrentHashMap.merge` to safely accumulate scores from multiple threads.
75. Implement a concurrent cache with `ConcurrentHashMap` that expires entries after 5 seconds.
76. Use `CopyOnWriteArrayList` as an event-listener registry — readers iterate while a writer adds.
77. Use `CountDownLatch` as a start gun for a simulated race between 5 threads.
78. Use `CountDownLatch` to wait for 3 services to initialise before starting the app.
79. Use `CyclicBarrier` to simulate 4 threads computing matrix rows in 3 phases.
80. Use `Phaser` where threads can deregister after completing their work.
81. Use `Semaphore(3)` to limit concurrent access to a simulated DB connection pool.
82. Use `Exchanger` to implement a double-buffer between a producer and consumer.

---

## Section 9 — ThreadLocal & Memory Model (83–90)

83. Use `ThreadLocal<SimpleDateFormat>` to share a date formatter safely across threads.
84. Demonstrate a `ThreadLocal` memory leak in a thread pool. Fix it with `remove()`.
85. Use `InheritableThreadLocal` and verify that child threads inherit the parent's value.
86. Write a program that demonstrates a visibility bug (flag not volatile). Then fix it.
87. Write a program that demonstrates a reordering bug (data published before flag). Fix with `volatile`.
88. Pad a `long[]` array to avoid false sharing. Measure performance improvement.
89. Use `LongAdder` vs `AtomicLong` under high contention and compare throughput.
90. Use `@Contended` (or manual padding) to eliminate false sharing in a counter array.

---

## Section 10 — Advanced & Production (91–100)

91. Use `ThreadMXBean` to detect a deadlock programmatically and print the thread names involved.
92. Capture a thread dump programmatically and filter for BLOCKED threads.
93. Write a `ThreadPoolTaskExecutor` configuration in Spring that names threads `order-worker-N`.
94. Implement a rate limiter using a `Semaphore` that refills 10 permits per second.
95. Implement a bulkhead: separate `ExecutorService` per downstream service.
96. Implement a circuit breaker pattern using `AtomicBoolean` and `ScheduledExecutorService`.
97. Create 10,000 virtual threads (Java 21), each sleeping 1s. Measure total time and memory.
98. Compare 10,000 platform threads vs 10,000 virtual threads for the same I/O-bound task.
99. Use `StructuredTaskScope.ShutdownOnFailure` to fetch user + orders in parallel, fail fast on error.
100. Implement a thread-safe LRU cache using `LinkedHashMap` + `ReadWriteLock`.

---

## Tips for Practice

- After writing each solution, intentionally break it and observe the failure mode
- Run with multiple JVM runs to observe non-deterministic behaviour
- Use `-ea` assertions to verify thread-safety invariants
- Profile with JFR after exercises 88–90 to see actual cache effects
