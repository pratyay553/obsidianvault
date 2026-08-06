# 84 - Glossary

Tags: #interview #section14
Related: [[83 - Revision Notes]] | [[85 - Cheat Sheet]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Definitions of every key term in the Java Multithreading vault — alphabetically ordered for fast lookup.

---

## A

**Atomic operation** — An operation that completes as a single, indivisible step. No other thread can observe it in a partially completed state. Examples: `AtomicInteger.incrementAndGet()`, a `volatile` read or write.

**ABA problem** — A CAS bug where a value changes from A → B → A; CAS sees A and assumes nothing changed, but the state may have changed in between. Solved with `AtomicStampedReference`.

---

## B

**BlockingQueue** — A thread-safe queue where `take()` blocks if empty and `put()` blocks if full. Used in producer-consumer patterns. Implementations: `ArrayBlockingQueue`, `LinkedBlockingQueue`, `SynchronousQueue`.

**BLOCKED (state)** — A thread waiting to acquire an intrinsic lock (`synchronized` block held by another thread).

**Busy-wait** — A thread continuously checks a condition in a loop without sleeping. Wastes CPU. Replace with `wait()`/`notify()` or `LockSupport.park()`.

---

## C

**CAS (Compare-And-Swap)** — A hardware instruction that atomically: reads a memory location, compares to an expected value, and writes a new value only if the comparison succeeded. The basis of all lock-free data structures.

**Cache line** — The unit of memory transfer between CPU cache and RAM, typically 64 bytes. Key to understanding false sharing.

**Callable** — A `@FunctionalInterface` whose `call()` method returns a value and can throw checked exceptions. Submitted to `ExecutorService.submit()`, returns a `Future`.

**CompletableFuture** — A non-blocking future that supports callback chaining (`thenApply`, `thenCompose`), combining (`allOf`, `anyOf`), and explicit completion.

**Concurrency** — Multiple tasks making progress, possibly interleaved on one or more cores. Does not require parallelism.

**Context switch** — The OS action of saving a running thread's state and restoring another thread's state to run next. Costs ~1–10µs + cache pollution.

**Critical section** — A code region that must execute atomically with respect to other threads — protected by a lock.

**CyclicBarrier** — A reusable synchronization point where N threads all wait for each other before proceeding together.

---

## D

**Daemon thread** — A background thread the JVM terminates automatically when all user threads finish. Set via `t.setDaemon(true)` before `start()`.

**Deadlock** — Two or more threads each hold a lock the other needs, forming a circular dependency. Neither can proceed.

**Double-checked locking** — A singleton pattern that checks a field twice (once without lock, once inside `synchronized`) to avoid locking on every call. Requires `volatile` on the field.

---

## E

**ExecutorService** — A framework for managing a pool of worker threads. Accepts `Runnable`/`Callable`, returns `Future`s, and provides controlled shutdown.

---

## F

**False sharing** — Two threads write to different variables that happen to share a CPU cache line. Each write invalidates the other core's copy of the line, causing severe performance degradation.

**ForkJoinPool** — A thread pool using work-stealing. Idle threads steal tasks from busy threads' queues. Optimised for recursive divide-and-conquer computation.

**Future** — A handle to the result of an async computation. Calling `get()` blocks until the result is ready.

---

## G

**GC thread** — Threads managed by the JVM to perform garbage collection. GC threads cause Stop-The-World pauses during which all application threads are frozen at safepoints.

---

## H

**Happens-before (HB)** — A formal relation in the Java Memory Model. If X happens-before Y, all writes made by X are guaranteed visible to Y. Established by: `volatile`, `synchronized`, `Thread.start()`/`join()`, `j.u.c` utilities.

---

## I

**Intrinsic lock (monitor)** — The lock associated with every Java object, used by `synchronized`. Reentrant; automatically released when `synchronized` block exits (including on exception).

**InheritableThreadLocal** — A `ThreadLocal` variant where child threads inherit the parent thread's values at creation time.

---

## J

**JMM (Java Memory Model)** — The specification defining how threads interact through memory. Defines visibility rules, ordering guarantees, and atomicity using the happens-before relation.

---

## L

**Livelock** — Threads keep changing state in response to each other but make no actual progress. High CPU, no BLOCKED state (unlike deadlock).

**Lock striping** — Splitting one lock into N locks, each protecting a subset of data. Reduces contention by N. Used internally by `ConcurrentHashMap`.

**LockSupport** — Low-level threading primitive providing `park()`/`unpark()`. Basis for `ReentrantLock`, `Semaphore`, and all `j.u.c` synchronizers.

---

## M

**Memory barrier (fence)** — A CPU/compiler instruction that prevents reordering of reads/writes across the barrier point. Inserted by the JVM for `volatile` and `synchronized`.

**Monitor** — Java's built-in locking mechanism (intrinsic lock). Every object has one. Used by `synchronized` keyword.

**Mutex** — Mutual exclusion lock. Only one thread may hold it at a time. `synchronized` and `ReentrantLock` are mutexes.

---

## P

**Parallelism** — Multiple tasks executing simultaneously on multiple CPU cores.

**Phaser** — A flexible barrier supporting dynamic thread registration/deregistration and multiple phases.

**Pinning (virtual threads)** — When a virtual thread executes `synchronized` code that blocks, it cannot unmount from its carrier OS thread. Degrades virtual thread scalability.

**Platform thread** — A traditional Java thread that maps 1:1 to an OS thread.

---

## R

**Race condition** — A bug where the outcome depends on the relative timing/ordering of thread execution. Two threads access shared mutable state without proper synchronization.

**ReentrantLock** — An explicit lock with features beyond `synchronized`: `tryLock()`, `lockInterruptibly()`, fair mode, multiple `Condition` objects.

**RUNNABLE (state)** — Thread is eligible to run (may be executing or waiting for CPU time).

---

## S

**Safepoint** — A location in code where the JVM can safely inspect/move objects (for GC). All threads must reach a safepoint before a Stop-The-World pause begins.

**Semaphore** — A counting lock with N permits. Threads acquire permits to proceed; block if none available. Used for rate limiting and bounded resource access.

**Spurious wakeup** — A thread waking from `wait()` without `notify()` being called. Always check the condition in a `while` loop, not `if`.

**Starvation** — A thread never gets CPU time or a lock because other threads always preempt it. Fixed with fair locks (`new ReentrantLock(true)`).

**Stop-The-World (STW)** — A GC phase where all application threads are paused at safepoints so GC can safely move objects.

**StampedLock** — An advanced lock with write, read, and optimistic-read modes. Not reentrant. Optimistic read: read without locking, validate stamp, retry if invalidated.

**Structured concurrency** — A model (Java 21+) where child task lifetimes are bound to an enclosing scope. Automatic cancellation of all children when scope exits.

---

## T

**TERMINATED (state)** — Thread's `run()` has returned or thrown an uncaught exception. Final state — cannot be restarted.

**ThreadLocal** — Provides per-thread variable copies. No synchronization needed. Must call `remove()` in thread pools.

**Thread pool** — A managed set of reusable threads. Tasks are submitted to a queue; pool threads pick them up. Avoids thread creation overhead.

**TIMED_WAITING (state)** — Thread voluntarily waiting with a maximum time: `sleep()`, `wait(ms)`, `join(ms)`.

---

## V

**Virtual thread (Project Loom)** — A JVM-managed lightweight thread (Java 21+). Multiplexed onto carrier OS threads. Cheap to create (~1KB stack); millions possible. Unmounts from carrier when blocking on I/O.

**Visibility problem** — A thread's write to a shared variable may not be seen by another thread due to CPU caching and compiler optimizations. Fixed with `volatile` or `synchronized`.

**Volatile** — A field modifier that ensures: (1) writes are immediately visible to all threads (via cache flush), (2) reads always fetch from main memory, (3) prevents reordering around the field access. Does NOT make compound operations atomic.

---

## W

**WAITING (state)** — Thread voluntarily waiting indefinitely: `wait()`, `join()`, `LockSupport.park()`. Exits only when notified/interrupted.

**Work stealing** — `ForkJoinPool` technique where idle threads steal tasks from the tail of busy threads' work queues. Improves utilisation for uneven workloads.
