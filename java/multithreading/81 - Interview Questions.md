# 81 - Interview Questions

Tags: #interview #section14
Related: [[83 - Revision Notes]] | [[85 - Cheat Sheet]] | [[84 - Glossary]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
A comprehensive bank of Java multithreading interview questions — grouped by difficulty and topic — with concise model answers.

---

## Beginner / Junior Level

**Q1: What is a thread?**
> The smallest unit of execution within a process. Threads share the process's heap but have their own stack and program counter.

**Q2: What is the difference between `start()` and `run()`?**
> `start()` creates a new OS thread and runs `run()` on it. Calling `run()` directly executes on the current thread — no new thread is created.

**Q3: What are the 6 Java thread states?**
> NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED (from `Thread.State` enum).

**Q4: What is the difference between `sleep()` and `wait()`?**
> `sleep()` pauses the current thread for a fixed time and does NOT release held locks. `wait()` releases the monitor lock and waits until `notify()` is called. `wait()` must be called inside a `synchronized` block.

**Q5: What is a daemon thread?**
> A background thread the JVM kills automatically when all user threads finish. Set with `t.setDaemon(true)` before `start()`.

**Q6: What is `volatile` used for?**
> Ensures visibility of writes across threads and prevents instruction reordering around the variable. Does NOT make compound operations atomic.

**Q7: What is the difference between `synchronized` method and `synchronized` block?**
> A synchronized method locks on `this` (instance) or the class object (static). A synchronized block lets you choose the lock object explicitly — finer granularity and shorter lock duration.

---

## Mid-Level

**Q8: What is a race condition?**
> When two threads access a shared variable simultaneously, at least one writes, and the outcome depends on scheduling order. Causes intermittent data corruption.

**Q9: What is a deadlock? How do you prevent it?**
> Two or more threads each hold a lock and wait for a lock held by the other — circular dependency. Prevent by: always acquiring locks in a consistent global order; using `tryLock(timeout)`; avoiding nested locks.

**Q10: What is the Java Memory Model?**
> A specification that defines when writes by one thread become visible to reads by another. Uses the happens-before relation. Without HB, writes may never be seen by other threads.

**Q11: What is the difference between `Runnable` and `Callable`?**
> `Runnable.run()` returns void; cannot throw checked exceptions. `Callable.call()` returns a typed value; can throw checked exceptions.

**Q12: How does `ConcurrentHashMap` differ from `Collections.synchronizedMap`?**
> `synchronizedMap` locks the entire map per operation. `ConcurrentHashMap` uses per-bin locking and CAS — allows many threads to read and write simultaneously without contention.

**Q13: What is `ThreadLocal` and when is it used?**
> Provides per-thread variable copies. Used for per-request state (security context, transaction, request ID) without synchronization. Always call `remove()` in thread pools.

**Q14: What is the difference between `CountDownLatch` and `CyclicBarrier`?**
> `CountDownLatch`: one-shot (non-reusable), one/few threads wait for many. `CyclicBarrier`: reusable, all N threads wait for each other at a barrier.

**Q15: What is `CompletableFuture`? How does it differ from `Future`?**
> `CompletableFuture` supports non-blocking callbacks (`.thenApply`, `.thenCompose`), combining multiple futures, and explicit completion. `Future` only supports blocking `.get()`.

**Q16: What is false sharing?**
> Two threads write to different variables that share a CPU cache line — each write invalidates the other's cache line. Fix: padding, `@Contended`, or `LongAdder`.

---

## Senior Level

**Q17: Explain the happens-before relation.**
> If X happens-before Y, all writes made by X are guaranteed visible to Y. Key HB rules: program order (within a thread), `volatile` write → read, `unlock` → `lock`, `Thread.start()` → child's actions, `Thread.join()` → joiner.

**Q18: What is double-checked locking and what is the fix?**
> An optimization for lazy singleton initialization. Without `volatile`, the JIT can publish the reference before the constructor completes. Fix: declare the instance field `volatile`.

**Q19: Explain `ReentrantLock` vs `synchronized`.**
> `ReentrantLock` adds: `tryLock()`, `lockInterruptibly()`, fair mode, `Condition` for multiple wait sets. `synchronized` is simpler and automatically releases on exit/exception. Prefer `synchronized` for simple cases.

**Q20: What is the difference between `wait()`/`notify()` and `LockSupport.park()`/`unpark()`?**
> `wait()`/`notify()` requires a `synchronized` block; `park()`/`unpark()` does not. `unpark()` can be called before `park()` (permit model — no signal loss). `unpark()` targets a specific thread; `notify()` is non-deterministic.

**Q21: What is a virtual thread (Java 21)? How does pinning affect it?**
> A JVM-managed lightweight thread multiplexed onto carrier OS threads. Cheap to create (millions). Pinning occurs when a virtual thread blocks inside a `synchronized` block — the carrier OS thread cannot be unmounted. Fix: use `ReentrantLock` for blocking operations.

**Q22: What is structured concurrency?**
> A model (Java 21+) where child tasks' lifetimes are bound to an enclosing scope. On scope exit, all child tasks are automatically cancelled. `StructuredTaskScope.ShutdownOnFailure` cancels all tasks if any fails; `ShutdownOnSuccess` cancels remaining when first succeeds.

**Q23: How do you size a thread pool?**
> CPU-bound: N ≈ CPU cores. I/O-bound: N ≈ cores × (1 + wait_time/service_time). Measure under realistic load — formula is a starting point.

**Q24: What is `StampedLock` and when is it used?**
> An advanced lock with three modes: write, read, and optimistic read. Optimistic read: read without locking, then validate the stamp — retries only on conflict. Not reentrant. Best for read-heavy workloads where writes are rare.

**Q25: How do you detect and diagnose a deadlock in production?**
> Thread dump: `jstack <pid>` — look for "Found one Java-level deadlock". Programmatically: `ThreadMXBean.findDeadlockedThreads()`. Prevention: consistent lock ordering; `tryLock(timeout)`.

---

## System Design Questions

**Q26: Design a rate limiter using Java concurrency primitives.**
> Use a `Semaphore` with N permits for N requests/second. OR use a token-bucket: `ScheduledExecutorService` refills a `AtomicInteger` every second; request threads decrement and reject if it reaches 0.

**Q27: You have a Spring Boot service timing out under load. How do you investigate?**
> 1. Check Tomcat thread pool saturation (Actuator `tomcat.threads.busy`). 2. Take thread dumps — look for all threads BLOCKED on same DB/HTTP resource. 3. JFR for lock contention analysis. 4. Check downstream service latency. 5. Review connection pool size (HikariCP `maximumPoolSize`).

**Q28: How would you implement a thread-safe LRU cache?**
> Use `LinkedHashMap` with `accessOrder=true` wrapped in `Collections.synchronizedMap`, or use `ConcurrentHashMap` with a `ConcurrentLinkedDeque` for LRU order. For production: use `Caffeine` cache — it uses striped lock + LRU/LFU internally.

---

## Cheat Sheet of Tricky Points

| Common trap | Truth |
|---|---|
| `t.run()` vs `t.start()` | `run()` = no new thread |
| `volatile` = atomic | ❌ Only for single reads/writes |
| `synchronized` releases on exception | ✅ Yes — always |
| `sleep()` releases locks | ❌ No |
| `wait()` releases locks | ✅ Yes |
| Thread pool threads are daemon | ❌ Default no — you must set it |
| More threads = faster | ❌ Only up to a point |
| `ConcurrentHashMap` allows null | ❌ No null keys or values |
