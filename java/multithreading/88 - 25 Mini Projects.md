# 88 - 25 Mini Projects

Tags: #practice #section15
Related: [[86 - 100 Coding Exercises]] | [[89 - 20 Production Design Problems]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
25 self-contained mini projects that demonstrate real-world use of Java concurrency — build each one end-to-end.

---

## Beginner Projects (1–8)

### Project 1 — Thread-Safe Counter Service
Build a `CounterService` with `increment()`, `decrement()`, `reset()`, and `get()`.  
Requirements: thread-safe; support 100 concurrent callers; no `synchronized` — use `AtomicLong`.  
Extension: add a `getAndReset()` that atomically returns and clears the value.

---

### Project 2 — Parallel File Word Count
Read a large text file. Split it into N chunks. Process each chunk on a thread pool (one thread per chunk). Merge results.  
Requirements: use `ExecutorService` + `Future<Map<String, Integer>>`. Merge maps in the main thread.  
Extension: use `ForkJoinPool` with recursive splitting.

---

### Project 3 — Background Heartbeat Monitor
A service sends a "heartbeat" to a remote endpoint every 5 seconds. The main app can start/stop the monitor.  
Requirements: daemon thread; graceful shutdown on `stop()`; handles `InterruptedException` correctly.

---

### Project 4 — Timed Task Executor
A simple `TaskScheduler` that runs tasks at a fixed rate.  
Requirements: Use `ScheduledExecutorService`. Support `schedule(task, delay)` and `scheduleRepeating(task, rate)`. Provide `cancel(taskId)`.

---

### Project 5 — Thread-Safe Event Bus
Publish events to multiple subscribers. Subscribers register/deregister dynamically.  
Requirements: `CopyOnWriteArrayList` for subscriber list; dispatch events on a thread pool; no blocking the publisher thread.

---

### Project 6 — Concurrent Download Manager
Download N files in parallel. Show progress percentage per file.  
Requirements: `ExecutorService`; `AtomicInteger` for progress per download; `CountDownLatch` to wait for all downloads; print summary when done.

---

### Project 7 — Pipeline with BlockingQueue
Three stages: Producer → Transformer → Consumer, each running on its own thread.  
Requirements: two `LinkedBlockingQueue`s connecting stages; graceful shutdown via poison pill; measure throughput.

---

### Project 8 — Rate-Limited API Client
Wrapper around an `HttpClient` that limits calls to 10 per second.  
Requirements: `Semaphore(10)` refilled every second by a `ScheduledExecutorService`; `tryAcquire` returns 429-like response when limit hit.

---

## Intermediate Projects (9–17)

### Project 9 — Thread-Safe LRU Cache
Fixed-capacity cache that evicts least-recently-used entry.  
Requirements: `LinkedHashMap(capacity, 0.75f, true)` + `ReadWriteLock`; `get` acquires read lock; `put` acquires write lock; configurable capacity.  
Extension: add TTL expiry via `ScheduledExecutorService`.

---

### Project 10 — Async HTTP Request Aggregator
Make 5 HTTP calls in parallel. Merge the results. Return to caller within 2 seconds or return partial results.  
Requirements: `CompletableFuture.allOf()` with timeout via `orTimeout(2, SECONDS)`; return whatever completed on timeout.

---

### Project 11 — Database Connection Pool
A bounded pool of simulated `Connection` objects.  
Requirements: `Semaphore(N)` for bounding; `ConcurrentLinkedQueue` for pool storage; `acquire()` blocks up to timeout; `release()` returns to pool; health check removes broken connections.

---

### Project 12 — Work-Stealing Task Executor
Implement a simplified work-stealing scheduler: N worker threads each have a `Deque<Task>`. When a worker's deque is empty, it steals from another's tail.  
Requirements: `ConcurrentLinkedDeque`; `AtomicBoolean running` flag for shutdown.

---

### Project 13 — Concurrent Merge Sort
Implement merge sort that splits at each level using `ForkJoinPool`.  
Requirements: `RecursiveAction` for in-place sort; `RecursiveTask<int[]>` for copy-based; measure speedup vs sequential for 10M elements.

---

### Project 14 — Actor-Style Message Processor
Each "actor" has a `LinkedBlockingQueue<Message>` and a dedicated thread.  
Requirements: `ActorRef.send(message)` puts in queue; actor thread processes messages sequentially; actors can send messages to each other; graceful shutdown propagation.

---

### Project 15 — Circuit Breaker
Wraps a `Supplier<T>` call. If it fails 5 times, open the circuit for 30 seconds.  
Requirements: states CLOSED → OPEN → HALF_OPEN; `AtomicReference<State>`; `ScheduledExecutorService` for half-open timer; thread-safe failure counter.

---

### Project 16 — Parallel Matrix Multiplication
Multiply two N×N matrices using a thread pool. Assign one thread per row of the result.  
Requirements: `ExecutorService`; `CountDownLatch` for completion; compare with sequential for 500×500 matrices.

---

### Project 17 — Real-Time Leaderboard
Concurrent leaderboard: many threads submit scores; top-10 query returns instantly.  
Requirements: `ConcurrentHashMap<String, AtomicLong>` for scores; `ConcurrentSkipListMap` for sorted view; `update(player, score)` and `getTop10()` both thread-safe; benchmark at 10k updates/sec.

---

## Advanced Projects (18–25)

### Project 18 — Thread-Safe Object Pool
Generic pool: `Pool<T>` backed by a `BlockingQueue<T>`.  
Requirements: configurable min/max size; `borrow()` blocks if empty; `return_(obj)` discards if pool full; `Factory<T>` interface for object creation; eviction of idle objects via scheduled check.

---

### Project 19 — Distributed Counter (Striped)
High-performance counter that eliminates false sharing by striping across 64 cells (pad each cell to 128 bytes).  
Requirements: `@Contended` or manual 7-long padding; hash thread ID to cell; `sum()` aggregates; compare to `AtomicLong` under 32-thread load.

---

### Project 20 — Async Result Cache (Memoisation)
First call to `compute(key)` starts an async computation. Subsequent calls with the same key return the same `CompletableFuture`.  
Requirements: `ConcurrentHashMap<Key, CompletableFuture<V>>`; `computeIfAbsent` ensures only one computation per key; handles exceptions (failed futures should be evicted and retried).

---

### Project 21 — Parallel Web Crawler
Crawl a website (simulated): fetch page → extract links → crawl links up to depth 3.  
Requirements: `ConcurrentHashMap<URL, Boolean>` for visited; `ForkJoinPool` for recursion; depth-limited `RecursiveTask`; rate limit to 5 concurrent fetches with `Semaphore`.

---

### Project 22 — Virtual Thread HTTP Server (Java 21)
Simple HTTP server using raw sockets where each accepted connection is handled by a virtual thread.  
Requirements: `ServerSocket.accept()` in a loop; `Thread.ofVirtual().start(() -> handleConnection(socket))`; handle 10,000 simultaneous connections; compare memory vs platform threads.

---

### Project 23 — Structured Concurrency Aggregator (Java 21)
Service that fetches user profile, order history, and recommendation list in parallel. Fails fast if any one fails.  
Requirements: `StructuredTaskScope.ShutdownOnFailure`; timeout the whole scope at 2 seconds; return partial result on timeout using `anyOf` fallback.

---

### Project 24 — Lock-Free Queue
Implement a single-producer / single-consumer lock-free ring buffer.  
Requirements: `AtomicInteger` for head/tail; `volatile` array; verify no locks; measure throughput vs `ArrayBlockingQueue`.

---

### Project 25 — Full Concurrency Benchmark Suite
Benchmark all the counter implementations: `synchronized`, `ReentrantLock`, `AtomicInteger`, `LongAdder`, striped custom counter.  
Requirements: JMH (Java Microbenchmark Harness) setup; vary thread counts 1, 2, 4, 8, 16, 32; generate throughput table; identify crossover points.

---

## Building Tips

- Start with a single-threaded implementation and verify correctness
- Then add threads and observe the failures (race conditions, deadlocks)
- Fix step by step, testing each fix
- Always add a stress test with many threads × many iterations
- Profile with JFR to find actual bottlenecks before optimising
