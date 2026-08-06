# 73 - Common Production Bugs

Tags: #jvm-internals #section12 #debugging
Related: [[36 - Deadlock]] | [[37 - Livelock]] | [[38 - Starvation]] | [[26 - Race Condition]] | [[70 - Debugging Multithreading]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Know the most common concurrency bugs in production Java systems — their symptoms, root causes, and fixes.

---

## Bug Catalogue

---

### Bug 1 — Race Condition

**Symptom**: Intermittent data corruption; wrong counts; occasional NPE on a supposedly initialised object.

**Root cause**: Two threads read-modify-write a shared variable without synchronization.
```java
// BROKEN
if (map.containsKey(key)) {
    map.get(key).process(); // another thread may remove key between check and get
}
```

**Fix**: Use atomic compound operations:
```java
map.computeIfPresent(key, (k, v) -> { v.process(); return v; });
```

---

### Bug 2 — Deadlock

**Symptom**: Application hangs indefinitely. Thread dump shows two threads each holding one lock and waiting for the other.

**Root cause**: Circular lock acquisition order.
```java
// Thread 1: lock A then B
// Thread 2: lock B then A  → deadlock
```

**Fix**: Always acquire locks in a consistent global order. Use `tryLock(timeout)` to detect and break deadlocks.

---

### Bug 3 — Thread Pool Exhaustion

**Symptom**: Requests time out or queue up; all pool threads are BLOCKED/WAITING in thread dump.

**Root cause**: Pool threads call a blocking operation (slow DB, external HTTP, sleep) and never return. Alternatively, pool size is too small for load.

**Fix**: Use separate pools for separate resources. Set timeouts on all blocking calls. Size pools correctly. Monitor queue depth.

---

### Bug 4 — Memory Leak via ThreadLocal

**Symptom**: Memory usage grows steadily; heap dumps show `ThreadLocalMap` entries accumulating.

**Root cause**: In a thread pool, calling `threadLocal.set()` without calling `remove()`. Thread pool threads never die — their `ThreadLocalMap` grows forever.

**Fix**: Always call `threadLocal.remove()` in a `finally` block.
```java
try {
    threadLocal.set(value);
    doWork();
} finally {
    threadLocal.remove(); // CRITICAL
}
```

---

### Bug 5 — Visibility Bug (Stale Read)

**Symptom**: A flag never seen as `true` by another thread; loop runs forever.

**Root cause**: Missing `volatile` — the flag is cached in the CPU register of the reading thread.
```java
boolean stop = false; // not volatile — Thread B may never see stop = true
```

**Fix**: Declare `volatile boolean stop = false;`

---

### Bug 6 — Incorrect Double-Checked Locking (pre-Java 5)

**Symptom**: Partially constructed singleton object visible to other threads — NPE on fields that look initialised.

**Root cause**: Without `volatile`, the JIT/CPU can reorder object construction: the reference can be published before the constructor finishes.
```java
// BROKEN (without volatile)
if (instance == null) {
    synchronized (lock) {
        if (instance == null) {
            instance = new Singleton(); // may see half-constructed object
        }
    }
}
```

**Fix**: Declare `private volatile static Singleton instance;`

---

### Bug 7 — Livelock

**Symptom**: High CPU usage, no progress, no thread in BLOCKED state (unlike deadlock).

**Root cause**: Two threads keep retrying a conflicting action, each backing off in response to the other.

**Fix**: Add random backoff jitter, or redesign to have one thread yield deterministically.

---

### Bug 8 — Missed Signal (notify before wait)

**Symptom**: Thread calls `wait()` and blocks forever even though the condition is already met.

**Root cause**: `notify()` was called before `wait()` — the signal is lost.
```java
// Thread A: condition = true; lock.notify();   ← signal lost if B hasn't called wait() yet
// Thread B: lock.wait();                        ← waits forever
```

**Fix**: Always check the condition in a `while` loop, and use `notifyAll()`. Consider `CountDownLatch` or `CompletableFuture` for one-shot events.

---

### Bug 9 — Executor Not Shut Down (Thread Leak)

**Symptom**: Thread count grows every time a code path is exercised; JVM never exits on unit tests.

**Root cause**: `Executors.newFixedThreadPool()` creates a pool but `shutdown()` is never called. Non-daemon pool threads keep the JVM alive.

**Fix**: Always shut down executors:
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
try {
    // submit tasks
} finally {
    pool.shutdown();
    pool.awaitTermination(30, TimeUnit.SECONDS);
}
```

---

### Bug 10 — Synchronized on Wrong Object

**Symptom**: Race condition persists despite `synchronized` keyword.

**Root cause**: Different threads synchronize on different object instances.
```java
synchronized (new Object()) { ... } // different object every call — NO mutual exclusion
```
Or: synchronizing on a `String` literal shared across classloaders.

**Fix**: Synchronize on a stable shared reference — a `private final` field, or `this` for single instance.

---

## Cheat Sheet

| Bug | Key symptom | Quick fix |
|---|---|---|
| Race condition | Intermittent data corruption | Atomic ops, synchronized |
| Deadlock | Hang, BLOCKED threads, circular lock in dump | Consistent lock order, tryLock |
| Pool exhaustion | Timeouts, all pool threads BLOCKED | Separate pools, timeouts, size correctly |
| ThreadLocal leak | Heap grows, ThreadLocalMap in dump | `remove()` in finally |
| Stale read | Loop never exits, stale flag | `volatile` |
| Bad DCL | NPE on singleton fields | `volatile` on instance field |
| Livelock | High CPU, no progress, not BLOCKED | Random backoff jitter |
| Missed signal | Wait blocks forever | Check condition in `while` loop |
| Thread leak | Thread count grows, JVM won't exit | `pool.shutdown()` always |
| Wrong lock object | synchronized doesn't help | Lock on `private final` field |
