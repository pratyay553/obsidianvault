# 85 - Cheat Sheet

Tags: #interview #section14
Related: [[83 - Revision Notes]] | [[84 - Glossary]] | [[81 - Interview Questions]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
The single-page reference for everything — print this out, stick it on your wall.

---

## Thread States

```
NEW → RUNNABLE ⇄ BLOCKED / WAITING / TIMED_WAITING → TERMINATED
```

| State | Caused by | Exits via |
|---|---|---|
| NEW | `new Thread()` | `start()` |
| RUNNABLE | `start()` | Preemption / blocking |
| BLOCKED | Waiting for `synchronized` lock | Lock released |
| WAITING | `wait()`, `join()`, `park()` | `notify()`, thread finishes, `unpark()` |
| TIMED_WAITING | `sleep(ms)`, `wait(ms)`, `join(ms)` | Timeout or notify |
| TERMINATED | `run()` returns | — |

---

## Key Method Comparisons

| Method | Releases lock? | Needs `synchronized`? | Timeout? | Throws |
|---|---|---|---|---|
| `sleep(ms)` | ❌ | ❌ | ✅ | `InterruptedException` |
| `wait()` | ✅ | ✅ | Optional | `InterruptedException` |
| `yield()` | ❌ | ❌ | ❌ | — |
| `join()` | ❌ | ❌ | Optional | `InterruptedException` |
| `park()` | ❌ | ❌ | Optional | — |

---

## Synchronization Primitives

| Tool | Atomic? | Reentrant? | Fair mode? | Key API |
|---|---|---|---|---|
| `synchronized` | ✅ | ✅ | ❌ | `wait`, `notify`, `notifyAll` |
| `ReentrantLock` | ✅ | ✅ | ✅ | `lock`, `tryLock`, `unlock`, `Condition` |
| `ReadWriteLock` | ✅ | ✅ | ✅ | `readLock()`, `writeLock()` |
| `StampedLock` | ✅ | ❌ | ❌ | `writeLock`, `readLock`, `tryOptimisticRead` |
| `volatile` | Read/write only | ❌ | ❌ | Field modifier |
| `AtomicInteger` | ✅ | ❌ | ❌ | `getAndIncrement`, `compareAndSet` |
| `LongAdder` | ✅ | ❌ | ❌ | `increment`, `sum` (high contention) |

---

## Executor & Task Types

| Type | Return | Throws checked? | Cancel? |
|---|---|---|---|
| `Runnable` | void | ❌ | N/A |
| `Callable<T>` | `T` | ✅ | Via `Future.cancel()` |
| `Future<T>` | `T` (blocking) | `ExecutionException` | `cancel(bool)` |
| `CompletableFuture<T>` | `T` (async) | `CompletionException` | `cancel(bool)` |

---

## Thread Pool Quick Reference

| Pool | Factory method | Use case | Risk |
|---|---|---|---|
| Fixed | `newFixedThreadPool(N)` | Steady load | Queue overflow |
| Cached | `newCachedThreadPool()` | Short bursty tasks | Thread explosion |
| Single | `newSingleThreadExecutor()` | Sequential background | N/A |
| Scheduled | `newScheduledThreadPool(N)` | Recurring tasks | N/A |
| ForkJoin | `ForkJoinPool.commonPool()` | CPU-bound parallel | Shared — don't block |
| Virtual | `newVirtualThreadPerTaskExecutor()` | I/O-bound (Java 21+) | Pinning |

---

## Concurrent Collections

| Collection | Reads | Writes | Null? | Best for |
|---|---|---|---|---|
| `ConcurrentHashMap` | Lock-free | Per-bin | ❌ | High-concurrency map |
| `CopyOnWriteArrayList` | Lock-free | O(n) copy | ✅ | Read-heavy list |
| `ArrayBlockingQueue` | Bounded | Lock | ❌ | Producer-consumer |
| `LinkedBlockingQueue` | Unbounded | Lock | ❌ | Producer-consumer |
| `ConcurrentLinkedQueue` | Lock-free | CAS | ❌ | Lock-free FIFO |

---

## Coordination Utilities

| Utility | Reusable? | Who waits | Key methods |
|---|---|---|---|
| `CountDownLatch` | ❌ | One/many wait for N | `countDown()`, `await()` |
| `CyclicBarrier` | ✅ | All N wait for each other | `await()` |
| `Phaser` | ✅ | Dynamic parties | `arriveAndAwaitAdvance()` |
| `Semaphore` | ✅ | N permits | `acquire()`, `release()` |
| `Exchanger` | ✅ | Exactly 2 swap | `exchange(V)` |

---

## Happens-Before Rules

| Rule | HB relation |
|---|---|
| Program order | Action A before B in same thread → A HB B |
| Monitor | `unlock(m)` → subsequent `lock(m)` |
| Volatile | `write(v)` → subsequent `read(v)` |
| Thread start | `t.start()` → any action in t |
| Thread join | Any action in t → `t.join()` returns |
| Transitivity | X HB Y and Y HB Z → X HB Z |

---

## Common Pitfalls & Fixes

| Pitfall | Wrong | Fix |
|---|---|---|
| `++` not atomic | `count++` | `AtomicInteger.incrementAndGet()` |
| `sleep` releases lock | Assumed yes | It does NOT |
| `notify` before `wait` | Lost signal | Use `while` loop + `notifyAll`, or `CountDownLatch` |
| `ThreadLocal` in pool | No `remove()` | `remove()` in `finally` |
| Self-invocation @Async | `this.asyncMethod()` | Inject self via Spring context |
| DCL without volatile | `singleton = new X()` | `private volatile static Singleton` |
| Wrong lock object | `synchronized(new Object())` | Lock on `private final` field |

---

## Virtual Threads (Java 21) Quick Ref

```java
// Create
Thread.ofVirtual().start(task);

// Pool (one VT per task)
Executors.newVirtualThreadPerTaskExecutor();

// Spring Boot 3.2+
spring.threads.virtual.enabled=true

// Detect pinning
-Djdk.tracePinnedThreads=full

// Fix pinning — replace synchronized with:
ReentrantLock lock = new ReentrantLock();
```

---

## Pool Sizing Formula

```
CPU-bound:   N = availableProcessors()
I/O-bound:   N = processors × (1 + wait_time / service_time)
```

---

## Debugging Commands

```bash
# Thread dump
jstack <pid>
jcmd <pid> Thread.print

# Find high-CPU thread
top -H -p <pid>          # get TID (decimal)
printf '%x\n' <TID>      # convert to hex
jstack <pid> | grep "nid=0x<hex>"  # find in dump

# JFR recording
jcmd <pid> JFR.start name=rec duration=60s filename=rec.jfr
```
