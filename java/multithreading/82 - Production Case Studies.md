# 82 - Production Case Studies

Tags: #interview #section14
Related: [[73 - Common Production Bugs]] | [[70 - Debugging Multithreading]] | [[36 - Deadlock]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Learn from real-world production concurrency incidents — the kind of stories interviewers love to hear and senior engineers have lived through.

---

## Case Study 1 — The Silent Counter Bug (Race Condition)

**System**: E-commerce order processing service  
**Symptom**: Daily report showed 98,742 orders processed but the database had 98,651 — a discrepancy of 91 orders. Appeared only under peak load.

**Root cause**:
```java
// Shared counter updated by 50 request threads
private int processedCount = 0;

public void processOrder(Order order) {
    save(order);
    processedCount++; // read-modify-write — not atomic!
}
```
`processedCount++` is three operations: read, increment, write. Under concurrency, two threads read the same value, both increment it, and one write is lost.

**Fix**:
```java
private final AtomicInteger processedCount = new AtomicInteger(0);
processedCount.incrementAndGet(); // atomic
```

**Lesson**: Always use `AtomicInteger` / `AtomicLong` for counters shared across threads. Never rely on `++` for shared state.

---

## Case Study 2 — The Midnight Deadlock (Scheduled Job)

**System**: Banking batch processor — nightly account reconciliation  
**Symptom**: Every night at 2 AM, the process hung indefinitely. CPU = 0%, no progress. Required manual restart.

**Root cause**: Thread dump revealed a deadlock:
```
Thread "reconciliation-1" waiting for lock on Account#4521 (held by "reconciliation-2")
Thread "reconciliation-2" waiting for lock on Account#1834 (held by "reconciliation-1")
```
Two threads were processing transfers between the same two accounts but in opposite order:
- Thread 1 locked Account A then tried to lock Account B
- Thread 2 locked Account B then tried to lock Account A

**Fix**: Always lock accounts in a consistent order (e.g., by account ID):
```java
long lower = Math.min(fromId, toId);
long higher = Math.max(fromId, toId);
synchronized (getAccount(lower)) {
    synchronized (getAccount(higher)) {
        performTransfer();
    }
}
```

**Lesson**: In any scenario with multiple locks, define a global lock ordering and enforce it everywhere.

---

## Case Study 3 — The Growing Heap (ThreadLocal Leak)

**System**: High-traffic REST API (Spring Boot + Tomcat)  
**Symptom**: Memory usage grew ~50MB per hour. Heap dumps showed `ThreadLocalMap` entries referencing `UserContext` objects.

**Root cause**:
```java
static ThreadLocal<UserContext> userContext = new ThreadLocal<>();

@Filter
public void doFilter(request, response, chain) {
    userContext.set(new UserContext(request)); // set context
    chain.doFilter(request, response);
    // MISSING: userContext.remove() — context lives in the pool thread forever
}
```
Tomcat reuses threads. Each request set a new `UserContext` in `ThreadLocal` but never removed the old one. Over time, 200 pool threads accumulated thousands of UserContext references each.

**Fix**:
```java
try {
    userContext.set(new UserContext(request));
    chain.doFilter(request, response);
} finally {
    userContext.remove(); // always remove in finally
}
```

**Lesson**: In thread pools, `ThreadLocal.remove()` in a `finally` block is mandatory, not optional.

---

## Case Study 4 — The Invisible Update (Visibility Bug)

**System**: Real-time trading engine  
**Symptom**: Occasionally, orders were sent based on stale price data even though the price had been updated milliseconds earlier.

**Root cause**:
```java
class PriceCache {
    private double latestPrice = 0.0; // not volatile

    void update(double price) { latestPrice = price; }
    double get() { return latestPrice; } // may return cached value
}
```
The price update thread wrote to `latestPrice`, but the order thread's CPU cached the old value in its L1 cache — it never re-read from main memory.

**Fix**:
```java
private volatile double latestPrice = 0.0;
```
The `volatile` write establishes happens-before with the subsequent volatile read, guaranteeing the new price is visible.

**Lesson**: Any field written by one thread and read by another must be `volatile` (or protected by `synchronized`) — even if writes and reads seem to "happen before" in wall-clock time.

---

## Case Study 5 — The Timeout Cascade (Thread Pool Exhaustion)

**System**: Microservice calling three downstream APIs  
**Symptom**: During a slow period for API #3, all requests to the microservice began timing out — even those that didn't need API #3 at all.

**Root cause**: All calls were made on the **same** Tomcat thread pool. When API #3 slowed to 30-second responses:
```
200 Tomcat threads all blocked waiting for API #3
→ Thread pool exhausted
→ Requests for API #1 and #2 also queued and timed out
```

**Fix**: Separate thread pools per downstream dependency (bulkhead pattern):
```java
@Bean ExecutorService api1Pool() { return Executors.newFixedThreadPool(20); }
@Bean ExecutorService api2Pool() { return Executors.newFixedThreadPool(20); }
@Bean ExecutorService api3Pool() { return Executors.newFixedThreadPool(20); }
// Slow API #3 only exhausts its own pool
```
Also: add circuit breakers (Resilience4j) and timeouts on all HTTP clients.

**Lesson**: Never share one thread pool for all external calls. Slow downstream services should not starve threads needed for healthy calls (bulkhead pattern).

---

## Case Study 6 — The Phantom Singleton (Broken Double-Checked Locking)

**System**: Configuration service  
**Symptom**: `NullPointerException` on `config.getProperty(key)` — `config` fields were null even though the singleton was not null.

**Root cause**:
```java
private static Config instance;  // NOT volatile

public static Config getInstance() {
    if (instance == null) {
        synchronized (Config.class) {
            if (instance == null) {
                instance = new Config(); // reference may be published before constructor finishes!
            }
        }
    }
    return instance;
}
```
The JIT reordered the write: published the reference to `instance` before the `Config` constructor completed, so another thread saw a non-null but partially constructed object.

**Fix**: `private static volatile Config instance;`

**Lesson**: Double-checked locking requires `volatile`. Or better: use initialization-on-demand holder pattern (no volatile needed):
```java
static class Holder { static final Config INSTANCE = new Config(); }
public static Config getInstance() { return Holder.INSTANCE; }
```

---

## Cheat Sheet — Production Incident Patterns

| Incident pattern | Signature | Fix |
|---|---|---|
| Race condition | Intermittent data corruption under load | Atomic ops / synchronization |
| Deadlock | Hang at specific time, circular lock in dump | Consistent lock ordering |
| ThreadLocal leak | Heap grows, `ThreadLocalMap` in dump | `remove()` in `finally` |
| Visibility bug | Stale reads, loop never exits | `volatile` |
| Thread pool exhaustion | All threads BLOCKED on same resource | Bulkhead: separate pools |
| Broken DCL | NPE on non-null singleton fields | `volatile` on instance field |
