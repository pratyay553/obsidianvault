# 87 - 50 Debugging Exercises

Tags: #practice #section15
Related: [[86 - 100 Coding Exercises]] | [[73 - Common Production Bugs]] | [[69 - Thread Dump Analysis]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
50 exercises where code is intentionally broken. Your task: identify the bug, explain why it happens, and fix it.

---

## Category A — Race Conditions (1–10)

**Exercise 1**
```java
class Counter {
    int count = 0;
    void increment() { count++; }
    int get() { return count; }
}
// 10 threads each call increment() 1000 times. Expected: 10000. Actual: less.
```
> Bug: `count++` is not atomic (read + increment + write). Fix: use `AtomicInteger` or `synchronized`.

---

**Exercise 2**
```java
List<String> results = new ArrayList<>();
// 5 threads each add 200 items concurrently
ExecutorService pool = Executors.newFixedThreadPool(5);
for (int i = 0; i < 1000; i++) pool.submit(() -> results.add(Thread.currentThread().getName()));
pool.shutdown(); pool.awaitTermination(10, TimeUnit.SECONDS);
System.out.println(results.size()); // sometimes < 1000, sometimes throws ConcurrentModificationException
```
> Bug: `ArrayList` is not thread-safe. Fix: use `CopyOnWriteArrayList` or `Collections.synchronizedList`.

---

**Exercise 3**
```java
Map<String, Integer> inventory = new HashMap<>();
// 10 threads check-then-update stock
if (inventory.getOrDefault(item, 0) > 0) {
    inventory.put(item, inventory.get(item) - 1); // sell one unit
}
// Items oversold — stock goes negative
```
> Bug: check-then-act is not atomic; race between `getOrDefault` and `put`. Fix: use `ConcurrentHashMap.compute()` with negative check.

---

**Exercise 4**
```java
static boolean initialized = false;
void init() {
    if (!initialized) {     // Thread A and B both see false
        setup();            // Both call setup()!
        initialized = true;
    }
}
```
> Bug: check-then-act race; `initialized` not volatile. Fix: use `synchronized` or `AtomicBoolean.compareAndSet()`.

---

**Exercise 5**
```java
// Lazy singleton — no volatile
static Config instance;
static Config get() {
    if (instance == null) {
        synchronized (Config.class) {
            if (instance == null) instance = new Config(); // JIT can reorder!
        }
    }
    return instance;
}
```
> Bug: missing `volatile` — another thread may see a partially constructed Config. Fix: `static volatile Config instance`.

---

**Exercise 6**
```java
int[] shared = {0};
Thread t1 = new Thread(() -> { for (int i=0; i<100000; i++) shared[0]++; });
Thread t2 = new Thread(() -> { for (int i=0; i<100000; i++) shared[0]++; });
t1.start(); t2.start(); t1.join(); t2.join();
System.out.println(shared[0]); // expected 200000, actual much less
```
> Bug: `shared[0]++` is not atomic. Fix: use `AtomicIntegerArray` or synchronize the update.

---

**Exercise 7**
```java
volatile int x = 0, y = 0;
// Thread A: x = 1; y = 1;
// Thread B: if (y == 1) assert x == 1; // can fail!
```
> Bug: `volatile` prevents reordering of each variable individually but doesn't order x relative to y. Fix: use `synchronized` or make only `y` volatile with program-order dependency from `x=1` to `y=1`.

---

**Exercise 8**
```java
Set<Integer> seen = new HashSet<>();
// 4 threads add random numbers to seen concurrently
pool.submit(() -> { for (int i=0; i<1000; i++) seen.add(random.nextInt()); });
// Result: missing elements, corrupted internal state, infinite loop in resize
```
> Bug: `HashSet` internal state is corrupted under concurrent modification. Fix: `Collections.synchronizedSet(new HashSet<>())` or `ConcurrentHashMap.newKeySet()`.

---

**Exercise 9**
```java
if (!cache.containsKey(key)) {
    cache.put(key, computeExpensive(key));
}
```
> Bug: even with `ConcurrentHashMap`, `containsKey` + `put` is not atomic — two threads may both compute the value. Fix: `cache.computeIfAbsent(key, k -> computeExpensive(k))`.

---

**Exercise 10**
```java
static int balance = 1000;
void transfer(int amount) {
    balance -= amount; // debit
    // ... network call ...
    balance += amount; // credit back on failure
}
```
> Bug: race condition on `balance`; debit and credit are separate non-atomic operations. Fix: use `AtomicInteger.addAndGet()` or synchronize the whole transfer.

---

## Category B — Deadlocks (11–20)

**Exercise 11**
```java
// Thread 1: synchronized(A) { synchronized(B) { ... } }
// Thread 2: synchronized(B) { synchronized(A) { ... } }
// Both threads block, waiting for the other
```
> Bug: circular lock acquisition. Fix: always lock A before B in both threads.

---

**Exercise 12**
```java
// Thread holds lock, then calls an external method that also tries to acquire the same lock
synchronized(this) {
    externalService.call(this); // externalService internally does synchronized(arg) — same lock!
}
```
> Bug: reentrant lock works here but alien method calling back can deadlock. Fix: don't pass `this` to external code while holding a lock on `this`.

---

**Exercise 13**
```java
ReentrantLock lock = new ReentrantLock();
// Thread A: lock.lock(); Thread.sleep(10000);
// Thread B: lock.lock(); // waits forever
```
> No deadlock but Thread B waits indefinitely. Fix: `lock.tryLock(5, TimeUnit.SECONDS)` with timeout handling.

---

**Exercise 14**
```java
// Philosopher 5 picks left then right (same as others) — deadlock
// All philosophers pick left chopstick, all wait for right
void eat(int id) {
    synchronized(chopstick[id]) {
        synchronized(chopstick[(id+1) % 5]) { eat(); }
    }
}
```
> Bug: circular dependency. Fix: last philosopher picks right then left; or use `tryLock` with random backoff.

---

**Exercise 15**
```java
// Database: Thread A holds row lock on user#1, waits for row lock on order#1
// Thread B holds row lock on order#1, waits for row lock on user#1
```
> Bug: DB-level deadlock. Fix: always access rows in consistent order (e.g., by primary key ascending); or use `SELECT FOR UPDATE` with timeout.

---

**Exercises 16–20**: Similar patterns — identify the circular dependency, draw the lock graph, and propose the ordering fix. Use `jstack` to confirm deadlock by looking for the "Found one Java-level deadlock" section.

---

## Category C — Visibility & Memory (21–30)

**Exercise 21**
```java
boolean stop = false; // not volatile
Thread t = new Thread(() -> { while (!stop) {} System.out.println("Stopped"); });
t.start();
Thread.sleep(1000);
stop = true; // Thread t may never see this
```
> Bug: `stop` is not volatile. Thread t's CPU may cache the value. Fix: `volatile boolean stop`.

---

**Exercise 22**
```java
int data = 0;
boolean ready = false; // not volatile
// Writer: data = 42; ready = true;
// Reader: while (!ready); System.out.println(data); // may see 0!
```
> Bug: no happens-before between writer and reader. Fix: `volatile boolean ready` establishes HB.

---

**Exercise 23**
```java
long counter = 0; // not volatile, not synchronized
// Thread A: counter = Long.MAX_VALUE;
// Thread B: long v = counter; // may see a half-written value (two 32-bit writes)
```
> Bug: 64-bit writes are not atomic on 32-bit JVMs. Fix: `volatile long counter` or `AtomicLong`.

---

**Exercises 24–30**: Variations on visibility bugs — `static` fields shared across threads, mutable objects published via non-volatile references, immutable classes published unsafely. Identify missing `volatile` or `synchronized` in each.

---

## Category D — Thread Pool Problems (31–40)

**Exercise 31**
```java
ExecutorService pool = Executors.newFixedThreadPool(4);
pool.submit(() -> doWork());
// No shutdown() called — JVM never exits in unit test
```
> Bug: non-daemon thread pool prevents JVM exit. Fix: call `pool.shutdown()` (and `awaitTermination` if needed) in a `finally` block.

---

**Exercise 32**
```java
ExecutorService pool = Executors.newCachedThreadPool();
for (int i = 0; i < 100_000; i++) pool.submit(() -> Thread.sleep(60_000));
// OutOfMemoryError: unable to create new native thread
```
> Bug: `newCachedThreadPool` creates unbounded threads. Fix: use `newFixedThreadPool(N)` or virtual threads.

---

**Exercise 33**
```java
// All 200 Tomcat threads call downstream API that takes 30s
// New requests get HTTP 503 — pool exhausted
```
> Bug: all threads blocked waiting on slow API. Fix: bulkhead pattern (separate pool per downstream), circuit breaker, or async/non-blocking HTTP client.

---

**Exercise 34**
```java
static ThreadLocal<Connection> conn = new ThreadLocal<>();
// In Tomcat filter:
conn.set(dataSource.getConnection()); // set per request
// Missing: conn.remove() in finally block
// After 1000 requests: heap full of Connection objects
```
> Bug: ThreadLocal leak in thread pool. Fix: `finally { conn.remove(); }`.

---

**Exercises 35–40**: Thread pool queue overflow (`LinkedBlockingQueue` at capacity → `RejectedExecutionException`); unhandled exceptions in `void` `@Async` methods; incorrect `ThreadFactory` that creates daemon threads for critical work; tasks submitted after `shutdown()`.

---

## Category E — Synchronization Mistakes (41–50)

**Exercise 41**
```java
synchronized (new Object()) { // different object every call!
    criticalSection();
}
```
> Bug: locking on a new object each time provides no mutual exclusion. Fix: lock on a `private final` field.

---

**Exercise 42**
```java
synchronized ("LOCK") { // String literal — may be shared across classloaders
    criticalSection();
}
```
> Bug: string literals are interned — shared across the JVM, causing unintended cross-module locking. Fix: use a dedicated `private static final Object LOCK = new Object()`.

---

**Exercise 43**
```java
// wait() called without checking condition first (using if, not while)
synchronized(this) {
    if (queue.isEmpty()) wait(); // spurious wakeup → queue still empty → NPE!
    process(queue.poll());
}
```
> Bug: spurious wakeups mean `wait()` can return without `notify()`. Fix: `while (queue.isEmpty()) wait()`.

---

**Exercise 44**
```java
// notify() used instead of notifyAll() in producer-consumer
// Two consumers waiting — only one is woken, the wrong one
```
> Bug: `notify()` wakes one arbitrary thread; if it's a consumer when a producer should wake, the signal is "wasted". Fix: use `notifyAll()` or separate `Condition`s for producers and consumers.

---

**Exercise 45**
```java
@Async
private void asyncMethod() { ... } // private!
```
> Bug: Spring AOP only intercepts public methods. `@Async` is silently ignored. Fix: make it `public`.

---

**Exercises 46–50**: `@Async` called from the same bean (self-invocation bypasses proxy); `CompletableFuture` exception swallowed (no `.exceptionally()` handler); `ForkJoinPool.commonPool()` used for blocking I/O tasks (starves parallel streams); `Semaphore.release()` called without matching `acquire()` (permits exceed initial count); `ConcurrentHashMap.size()` used for critical business logic (it's approximate).

---

## Debugging Methodology Reminder
1. Read the exception and stack trace carefully
2. Identify shared mutable state
3. Find all threads accessing that state
4. Check for missing synchronization (volatile, synchronized, atomic)
5. Check for wrong lock object or wrong lock scope
6. Use `jstack` for BLOCKED/WAITING states
7. Use JFR for lock contention and CPU analysis
