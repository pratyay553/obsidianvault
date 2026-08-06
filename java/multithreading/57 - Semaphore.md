# 57 - Semaphore

Tags: #coordination-utilities #section9
Related: [[32 - ReentrantLock]] | [[54 - CountDownLatch]] | [[44 - Thread Pool]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `Semaphore` — a counting lock that controls how many threads can access a resource simultaneously.

---

## Beginner Explanation
A `Semaphore` holds a number of **permits**. A thread must acquire a permit to proceed. If no permits are available, it blocks until another thread releases one. Use it to limit concurrent access (e.g., max 10 threads in a database connection pool).

---

## Analogy
A parking lot with N spaces. Each car (thread) must find a free space (acquire a permit) to park. When a car leaves (releases permit), one waiting car can enter. The parking lot has a fixed capacity at any moment.

---

## Technical Deep Dive

### API
```java
Semaphore sem = new Semaphore(N);          // N permits
Semaphore sem = new Semaphore(N, true);    // fair — FIFO order

sem.acquire();                   // take 1 permit (blocks if unavailable)
sem.acquire(n);                  // take n permits
sem.tryAcquire();                // non-blocking attempt; returns boolean
sem.tryAcquire(t, TimeUnit);     // try with timeout
sem.release();                   // return 1 permit
sem.release(n);                  // return n permits
sem.availablePermits();          // current permit count
sem.getQueueLength();            // approx threads waiting
```

### Key behaviours
- **Binary semaphore** (`new Semaphore(1)`) ≈ a mutex, but unlike `ReentrantLock` it can be released by a **different thread** than the one that acquired it
- **Fairness**: `new Semaphore(N, true)` guarantees FIFO ordering — prevents starvation but lowers throughput
- **Permits can exceed initial value**: `release()` without a prior `acquire()` increases permits above N (unlike locks)
- **Not reentrant**: same thread calling `acquire()` twice consumes two permits

### Semaphore vs Lock

| Aspect | Semaphore | ReentrantLock |
|---|---|---|
| Max concurrent holders | N | 1 |
| Cross-thread release | ✅ Yes | ❌ No |
| Reentrancy | ❌ No | ✅ Yes |
| Use case | Rate limiting, pools | Mutual exclusion |

---

## Code

```java
import java.util.concurrent.*;

public class SemaphoreDemo {
    // Limit concurrent DB connections to 3
    static final Semaphore dbPool = new Semaphore(3, true);

    static void simulateDbWork(int threadId) throws InterruptedException {
        System.out.println("Thread " + threadId + " waiting for DB connection...");
        dbPool.acquire();
        try {
            System.out.println("Thread " + threadId + " GOT connection (available: "
                               + dbPool.availablePermits() + ")");
            Thread.sleep(500); // simulate DB work
        } finally {
            dbPool.release();
            System.out.println("Thread " + threadId + " released connection");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(8);
        for (int i = 1; i <= 8; i++) {
            final int id = i;
            pool.submit(() -> {
                try { simulateDbWork(id); }
                catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            });
        }
        pool.shutdown();
        pool.awaitTermination(15, TimeUnit.SECONDS);

        // tryAcquire — non-blocking
        Semaphore rateLimiter = new Semaphore(2);
        for (int i = 0; i < 5; i++) {
            if (rateLimiter.tryAcquire()) {
                System.out.println("Request " + i + " accepted");
                // must release when done
                rateLimiter.release();
            } else {
                System.out.println("Request " + i + " rejected — rate limit");
            }
        }
    }
}
```

---

## Interview Questions

**Q: What is the difference between a `Semaphore` and a `ReentrantLock`?**
> A `ReentrantLock` allows only one thread at a time and is reentrant (same thread can lock again). A `Semaphore` allows up to N threads simultaneously and can be released by a different thread than the one that acquired it.

**Q: What is a binary semaphore? How does it differ from a mutex?**
> A binary semaphore (`new Semaphore(1)`) allows one thread at a time but can be released by any thread. A mutex (like `ReentrantLock`) enforces that only the owning thread can release it.

**Q: Can `availablePermits()` go negative?**
> No — `acquire()` blocks when permits reach 0. But multiple `release()` calls without `acquire()` can raise it above the initial value.

---

## Cheat Sheet
| Method | Description |
|---|---|
| `acquire()` | Take permit (blocks if none) |
| `tryAcquire()` | Non-blocking acquire |
| `release()` | Return permit |
| `availablePermits()` | Current free permits |
| Fair mode | `new Semaphore(N, true)` — FIFO |
| Use for | Rate limiting, connection pools, bounded access |
