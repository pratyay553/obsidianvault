# 89 - 20 Production Design Problems

Tags: #practice #section15
Related: [[82 - Production Case Studies]] | [[88 - 25 Mini Projects]] | [[81 - Interview Questions]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
20 system-design problems focused on concurrency — the kind asked in senior engineering interviews and encountered in real production systems. For each: design the solution, identify race conditions, and justify your synchronization choices.

---

## Problem 1 — Thread-Safe Singleton with Lazy Init
**Question**: Design a thread-safe lazy singleton that is initialized only once, even if 1000 threads call `getInstance()` simultaneously.  
**Constraints**: No synchronization on every `getInstance()` call.

**Solution approach**:
- Option A: initialization-on-demand holder (classloading guarantees single init, no `volatile` needed)
- Option B: double-checked locking with `volatile`
- Option C: `enum` singleton (thread-safe, serialisation-safe)

```java
// Best: holder pattern — no volatile, no synchronized on hot path
class Singleton {
    private Singleton() {}
    private static class Holder { static final Singleton INSTANCE = new Singleton(); }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}
```

---

## Problem 2 — High-Throughput Request Counter
**Question**: Design a counter for HTTP requests that is incremented on every request (10k req/s) and read every 5 seconds for a metrics dashboard.  
**Constraints**: Writes must be nearly lock-free. Reads can be slightly stale.

**Solution**: `LongAdder` — striped per-CPU cells, near-zero contention on writes; `sum()` for reads aggregates cells. Better than `AtomicLong` under high contention.

---

## Problem 3 — Async Order Processing Pipeline
**Question**: An order comes in. You must simultaneously: (a) validate payment, (b) check inventory, (c) send confirmation email. All must succeed before marking order complete.

**Solution**:
```java
CompletableFuture<PaymentResult> payment = validatePayment(order);
CompletableFuture<InventoryResult> inventory = checkInventory(order);
CompletableFuture<Void> email = sendEmail(order);

CompletableFuture.allOf(payment, inventory, email)
    .thenRun(() -> markComplete(order))
    .exceptionally(ex -> { rollback(order); return null; });
```
Use `allOf` for parallel fan-out. Handle individual failures with `exceptionally` per future before the `allOf`.

---

## Problem 4 — Database Connection Pool
**Question**: Design a thread-safe connection pool: `getConnection()` blocks if all N connections are in use; `releaseConnection()` returns one to the pool.

**Solution**:
- `Semaphore(N)` controls access — `acquire()` in `getConnection()`, `release()` in `releaseConnection()`
- `ConcurrentLinkedQueue<Connection>` stores available connections
- `releaseConnection()` checks health before returning; discards and creates new if broken
- Timeout variant: `tryAcquire(timeout, unit)` → `TimeoutException` if no connection in time

---

## Problem 5 — Bulkhead: Isolate Downstream Dependencies
**Question**: Your service calls 3 external APIs: UserService, InventoryService, PaymentService. PaymentService sometimes becomes slow. Design so a slow PaymentService doesn't starve UserService threads.

**Solution**: Separate `ExecutorService` (or virtual thread pool) per dependency:
```java
ExecutorService userPool      = Executors.newFixedThreadPool(20);
ExecutorService inventoryPool = Executors.newFixedThreadPool(10);
ExecutorService paymentPool   = Executors.newFixedThreadPool(10);
```
A timeout on paymentPool threads never affects userPool. Set timeouts on all HTTP clients.

---

## Problem 6 — Concurrent Cache with Expiry
**Question**: Design a cache where entries expire after 60 seconds. `get(key)` returns null for expired entries. Multiple threads call `get` and `put` concurrently.

**Solution**:
```java
record Entry<V>(V value, long expiresAt) {}
ConcurrentHashMap<K, Entry<V>> map = new ConcurrentHashMap<>();

V get(K key) {
    Entry<V> e = map.get(key);
    if (e == null || e.expiresAt() < System.currentTimeMillis()) {
        map.remove(key); return null;
    }
    return e.value();
}
void put(K key, V value) {
    map.put(key, new Entry<>(value, System.currentTimeMillis() + 60_000));
}
```
Add a `ScheduledExecutorService` to sweep expired entries periodically.

---

## Problem 7 — Fan-Out + Fan-In Aggregator
**Question**: Given a list of 100 product IDs, fetch prices from a pricing service in parallel. Return all results within 3 seconds; return cached/default price for any that time out.

**Solution**:
```java
List<CompletableFuture<Price>> futures = ids.stream()
    .map(id -> CompletableFuture
        .supplyAsync(() -> fetchPrice(id), pool)
        .completeOnTimeout(DEFAULT_PRICE, 3, TimeUnit.SECONDS)
        .exceptionally(ex -> DEFAULT_PRICE))
    .toList();

CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
return futures.stream().map(CompletableFuture::join).toList();
```

---

## Problem 8 — Leader Election (Single JVM)
**Question**: Multiple threads compete to be the "leader" that runs a background job. Only one should run at a time.  
**Constraints**: Threads may start/stop; leadership should transfer if leader thread dies.

**Solution**: `AtomicReference<Thread>` for current leader. Each candidate thread checks with `compareAndSet(null, currentThread)`. On leadership loss (interrupt/crash), successor wins next `compareAndSet`. Heartbeat with scheduled check.

---

## Problem 9 — Task Deduplication
**Question**: Thousands of events arrive per second. If the same event key arrives multiple times within 100ms, process it only once.

**Solution**:
```java
ConcurrentHashMap<String, Long> recentKeys = new ConcurrentHashMap<>();
// Scheduled cleanup every 200ms: remove entries older than 100ms

void handle(Event event) {
    long now = System.currentTimeMillis();
    Long prev = recentKeys.putIfAbsent(event.key(), now);
    if (prev == null || now - prev > 100) {
        recentKeys.put(event.key(), now); // update timestamp
        process(event);
    }
    // else: duplicate — skip
}
```

---

## Problem 10 — Write-Ahead Log Flusher
**Question**: Multiple threads append to an in-memory log. A background thread flushes to disk every 50ms. Design for maximum write throughput.

**Solution**:
- Double buffer: threads write to `activeBuffer`; flusher swaps buffers atomically with `AtomicReference<Buffer>`
- Writers: `buffer.get().append(entry)` — concurrent appends with per-buffer lock
- Flusher: `Buffer old = buffer.getAndSet(new Buffer()); old.flush();`
- No writer blocks on disk I/O

---

## Problem 11 — Ordered Event Processing
**Question**: Events arrive out of order from multiple producers. Process events in sequence number order (1, 2, 3, …) with no gaps.

**Solution**:
- `ConcurrentSkipListMap<Long, Event>` for ordering
- `AtomicLong nextExpected` tracks next sequence number
- When event N arrives: `map.put(N, event)`; drain loop while `map.containsKey(nextExpected)`: process and increment
- Drain loop must be synchronized to prevent two threads processing the same sequence

---

## Problem 12 — Graceful Server Shutdown
**Question**: Design shutdown logic: stop accepting new requests; wait for in-flight requests to complete (up to 30 seconds); force-close remaining.

**Solution**:
```java
ExecutorService pool = Executors.newFixedThreadPool(200);

// On shutdown signal:
pool.shutdown(); // stop accepting new tasks
if (!pool.awaitTermination(30, TimeUnit.SECONDS)) {
    pool.shutdownNow(); // interrupt remaining
    pool.awaitTermination(5, TimeUnit.SECONDS);
}
```
Add `CountDownLatch` or `Semaphore` to track in-flight request count for more precise control.

---

## Problem 13 — Concurrent Inventory Reservation
**Question**: 100 users try to buy the last 10 units of a product simultaneously. No overselling allowed.

**Solution**:
```java
AtomicInteger stock = new AtomicInteger(10);

boolean reserve(int qty) {
    return stock.updateAndGet(s -> s >= qty ? s - qty : s) != stock.get();
    // Better:
    int current;
    do {
        current = stock.get();
        if (current < qty) return false; // out of stock
    } while (!stock.compareAndSet(current, current - qty));
    return true;
}
```

---

## Problem 14 — Read-Heavy Configuration Store
**Question**: A configuration service stores 1000 key-value pairs. Reads happen 100,000x/sec; writes (updates) happen once per minute.

**Solution**: `ReadWriteLock` — `readLock()` for all `get()` calls; `writeLock()` for updates. Or: `CopyOnWriteHashMap` simulation — replace the entire immutable map reference on write (`volatile Map<K,V> config`; on update: create new map, assign atomically). Readers always see a consistent snapshot.

---

## Problem 15 — Parallel Test Runner
**Question**: Run 500 test cases in parallel on a thread pool sized to CPU cores. Collect pass/fail results. Print a summary when all complete.

**Solution**:
```java
ExecutorService pool = Executors.newFixedThreadPool(cores);
List<Future<TestResult>> futures = tests.stream()
    .map(test -> pool.submit(() -> runTest(test)))
    .toList();
pool.shutdown();
pool.awaitTermination(5, TimeUnit.MINUTES);

long passed = futures.stream()
    .map(f -> { try { return f.get(); } catch (Exception e) { return FAILED; } })
    .filter(TestResult::isPassed).count();
```

---

## Problem 16 — Hot Cache Reload
**Question**: A cache holds product data. Every 5 minutes, reload all data from DB. During reload, reads should not see a partially updated cache.

**Solution**: Atomic reference swap. Build new cache in background on fresh `HashMap`. On completion: `cacheRef.set(newCache)` — single atomic write to a `volatile` reference. Readers always see a complete consistent snapshot; no reader is ever blocked.

---

## Problem 17 — Saga Orchestrator
**Question**: A distributed transaction spans 3 services. If any step fails, run compensating transactions for completed steps.

**Solution**:
```java
CompletableFuture<Void> step1 = service1.execute(order)
    .exceptionally(ex -> { /* compensate1 */ throw propagate(ex); });
CompletableFuture<Void> step2 = step1.thenCompose(__ -> service2.execute(order))
    .exceptionally(ex -> { compensate1(); throw propagate(ex); });
CompletableFuture<Void> step3 = step2.thenCompose(__ -> service3.execute(order))
    .exceptionally(ex -> { compensate2(); compensate1(); throw propagate(ex); });
```

---

## Problem 18 — Thread Pool Monitoring
**Question**: Design a monitoring system that alerts when a thread pool is more than 80% saturated.

**Solution**:
```java
ThreadPoolExecutor pool = (ThreadPoolExecutor) Executors.newFixedThreadPool(100);

// Scheduled check every 10 seconds:
scheduler.scheduleAtFixedRate(() -> {
    double utilisation = (double) pool.getActiveCount() / pool.getMaximumPoolSize();
    if (utilisation > 0.8) alerting.send("Pool at " + (int)(utilisation * 100) + "%");
}, 0, 10, TimeUnit.SECONDS);
```
Also expose via Actuator: `tomcat.threads.busy` / `tomcat.threads.config.max`.

---

## Problem 19 — Virtual Thread Adoption Strategy (Java 21)
**Question**: You have an existing Spring Boot service with 50 `@Async` methods and a Tomcat server. How do you migrate to virtual threads with minimum risk?

**Solution steps**:
1. Update to Spring Boot 3.2+ and Java 21
2. Set `spring.threads.virtual.enabled=true` — Tomcat and `@Async` use virtual threads
3. Audit all `synchronized` blocks that contain blocking I/O → replace with `ReentrantLock`
4. Enable `-Djdk.tracePinnedThreads=full` in staging — detect pinning
5. Load test at 10× normal concurrency — virtual threads should handle without pool exhaustion
6. Monitor memory: virtual threads use less but watch for `ThreadLocal` misuse

---

## Problem 20 — Designing for Zero Downtime Deployment
**Question**: How do you design a Java service so it can be restarted with zero dropped requests?

**Solution**:
- **Graceful shutdown**: `ExecutorService.shutdown()` + `awaitTermination(30s)` on SIGTERM
- **In-flight tracking**: `AtomicInteger requestsInFlight`; increment on accept, decrement on complete
- **Drain**: on shutdown signal, stop accepting, wait for `requestsInFlight == 0` or timeout
- **Load balancer**: health check `/actuator/health` returns DOWN during drain → LB routes to new instance
- **Connection draining**: set `server.shutdown=graceful` in Spring Boot application.yml
