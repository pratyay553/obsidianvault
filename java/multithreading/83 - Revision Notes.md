# 83 - Revision Notes

Tags: #interview #section14
Related: [[81 - Interview Questions]] | [[85 - Cheat Sheet]] | [[84 - Glossary]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Quick-revision summaries of every section — read this the night before an interview.

---

## Section 1–3 — Foundations

- **Thread** = smallest execution unit; shares heap with siblings; own stack + PC
- **Process** = own address space; threads are lightweight units within it
- Multithreading needed for: multi-core CPU utilisation, I/O overlap (concurrency), responsiveness
- **Concurrency** ≠ **Parallelism**: concurrent = interleaved on 1+ cores; parallel = simultaneous on multiple cores

---

## Section 2 — Creating Threads

| Method | Key point |
|---|---|
| `extends Thread` | Inflexible — single inheritance used |
| `implements Runnable` | Preferred — separates task from thread |
| Lambda | `new Thread(() -> ...)` — concise |
| `ExecutorService` | Production standard — thread pools |
| `Callable` + `Future` | When you need a return value |

- Always call `t.start()`, never `t.run()`
- `Callable.call()` can throw checked exceptions; `Runnable.run()` cannot

---

## Section 3 — Thread Lifecycle

**6 states**: NEW → RUNNABLE → {BLOCKED | WAITING | TIMED_WAITING} → TERMINATED

- **BLOCKED**: waiting for `synchronized` lock
- **WAITING**: `wait()`, `join()`, `park()` — needs notification
- **TIMED_WAITING**: `sleep()`, `wait(ms)`, `join(ms)` — auto-wakes
- Cannot restart a TERMINATED thread

---

## Section 3 — Control Methods

| Method | Releases lock? | Interruptible? |
|---|---|---|
| `sleep(ms)` | ❌ No | ✅ Yes |
| `wait()` | ✅ Yes | ✅ Yes |
| `yield()` | ❌ N/A | ❌ No |
| `join()` | ❌ N/A | ✅ Yes |

- `interrupt()` sets a flag; `sleep/wait/join` throw `InterruptedException`
- Always restore interrupt flag: `Thread.currentThread().interrupt()`

---

## Section 4 — Synchronization

- **Race condition**: shared mutable state + concurrent access without sync
- **`synchronized`**: intrinsic lock; reentrant; releases on exception
- **`volatile`**: visibility + ordering; NOT atomicity for compound ops
- **`AtomicXxx`**: lock-free atomics using CAS; prefer `LongAdder` for high-contention counters
- **CAS**: compare-and-swap; retry loop on failure; basis of all lock-free structures

---

## Section 5 — Advanced Locks

| Lock | Key feature |
|---|---|
| `ReentrantLock` | `tryLock()`, `lockInterruptibly()`, fair mode, `Condition` |
| `ReadWriteLock` | Multiple concurrent readers OR one exclusive writer |
| `StampedLock` | Optimistic read mode (no lock — validate stamp) |
| `Condition` | Multiple wait sets per lock (vs one per object monitor) |

---

## Section 6–7 — Problems & Communication

- **Deadlock**: circular lock dependency → consistent lock ordering, `tryLock(timeout)`
- **Livelock**: threads keep retrying without progress → random backoff
- **Starvation**: low-priority thread never runs → fair locks
- **Producer-Consumer**: use `BlockingQueue` — blocks on empty (consumer) and full (producer)
- `wait()`/`notify()` → always in `while` loop (spurious wakeups), always inside `synchronized`

---

## Section 8 — Thread Pools & Executors

- **`ExecutorService`**: submit tasks, get `Future`s, shutdown when done
- **`FixedThreadPool(N)`**: N workers; queue unbounded (danger: OOM under load)
- **`CachedThreadPool`**: unbounded threads; danger: thread explosion
- **`ScheduledExecutorService`**: fixed-rate / fixed-delay scheduling
- **`ForkJoinPool`**: work-stealing; ideal for recursive CPU-bound tasks
- **`CompletableFuture`**: async pipeline; `thenApply`, `thenCompose`, `allOf`, `anyOf`
- **`ParallelStream`**: uses `ForkJoinPool.commonPool()`; avoid for I/O tasks

---

## Section 9 — Concurrent Collections & Coordination

| Utility | One-liner |
|---|---|
| `ConcurrentHashMap` | Lock-free reads; per-bin writes; no null |
| `CopyOnWriteArrayList` | Copy on write; great for read-heavy |
| `CountDownLatch` | One-shot gate; threads wait for count→0 |
| `CyclicBarrier` | Reusable barrier; all N threads meet |
| `Phaser` | Flexible; dynamic parties; multiple phases |
| `Semaphore` | N permits; rate limiting, connection pools |
| `Exchanger` | Exactly 2 threads swap objects |
| `LockSupport` | Low-level park/unpark; basis of all locks |

---

## Section 10 — ThreadLocal

- Per-thread variable copy; no synchronization needed
- **MUST call `remove()` in `finally`** when using with thread pools
- `InheritableThreadLocal`: child threads inherit parent's value at creation
- Java 21+: prefer `ScopedValue` for virtual threads

---

## Section 11 — Java Memory Model

- JMM defines **happens-before (HB)** relation
- HB rules: program order, `volatile` write→read, `unlock`→`lock`, `start()`→child, `join()`→joiner
- Without HB: stale reads, reordering, 64-bit non-atomicity
- **False sharing**: different vars on same 64-byte cache line → fix with `@Contended` / `LongAdder`

---

## Section 12 — JVM Debugging

- **Thread dump**: `jstack <pid>` — look for deadlock, BLOCKED threads, hot RUNNABLE frames
- **JFR**: `jcmd <pid> JFR.start` — 1% overhead; lock contention, CPU, GC
- **High CPU thread**: `top -H -p <pid>` → TID → `printf '%x' TID` → `nid=0x<hex>` in jstack
- **Pool exhaustion**: all threads BLOCKED in dump; `tomcat.threads.busy == max`

---

## Section 13 — Frameworks

- **Spring MVC**: one Tomcat thread per request (default 200 threads)
- **`@Async`**: runs on `TaskExecutor`; must be public; no self-invocation
- **Spring security**: `ThreadLocal`-based; does not auto-propagate to `@Async`
- **Virtual threads (Java 21+)**: `spring.threads.virtual.enabled=true` — unlimited I/O concurrency
- **Kafka consumer**: one consumer per thread; max parallelism = partition count
- **Reactive (WebFlux)**: N event-loop threads (N=cores); non-blocking callbacks

---

## Section 14–15 — Interview tips

1. For every concurrency answer: mention **visibility**, **atomicity**, and **ordering** — the three axes of thread safety
2. Common traps: `++` not atomic; `sleep` doesn't release lock; `notify` before `wait` = lost signal
3. When asked about performance: profile first, then fix — `LongAdder` > `AtomicLong` under contention
4. Production story format: Symptom → Root cause → Fix → Prevention
