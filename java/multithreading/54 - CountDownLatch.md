# 54 - CountDownLatch

Tags: #coordination-utilities #section9
Related: [[55 - CyclicBarrier]] | [[56 - Phaser]] | [[57 - Semaphore]] | [[46 - ExecutorService]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `CountDownLatch` — a one-shot synchronization gate that lets one or more threads wait until a set of operations completes.

---

## Beginner Explanation
`CountDownLatch` is initialized with a count. Threads call `countDown()` to decrement it. Any thread that calls `await()` blocks until the count reaches zero. Once it hits zero, **all waiting threads are released and the latch can never be reused**.

---

## Analogy
A rocket launch: the countdown starts at 10. Multiple systems (threads) each confirm readiness by counting down (calling `countDown()`). The launch controller (`await()`) waits until zero before firing the engines.

---

## Technical Deep Dive

### API

```java
CountDownLatch latch = new CountDownLatch(N);

// In N worker threads:
latch.countDown(); // decrement; when reaches 0, release all waiters

// In waiting thread(s):
latch.await();                         // block until count == 0
latch.await(5, TimeUnit.SECONDS);      // block up to 5s, returns false on timeout
latch.getCount();                      // current count (for monitoring)
```

### Key properties
- **One-shot**: cannot be reset. Once count reaches 0, it stays 0. All future `await()` calls return immediately.
- **Multiple waiters**: any number of threads can call `await()` — all are released when count hits 0.
- **Multiple countdown callers**: any number of threads can call `countDown()`.
- **Non-cancellable count**: you cannot "count up" or reset.

### CountDownLatch vs CyclicBarrier

| Aspect | CountDownLatch | CyclicBarrier |
|---|---|---|
| Reusable | No | Yes |
| Who waits | One/few threads wait for many | All threads wait for each other |
| Count-up | No | N/A (barrier trips when all arrive) |
| Callback on trip | No | Optional `Runnable` |

### Two classic patterns

**Pattern 1 — Wait for N workers to finish:**
```
Latch(N) → N workers each countDown() → coordinator await()
```

**Pattern 2 — Start gun for N workers simultaneously:**
```
Latch(1) → N workers await() → coordinator countDown() → all start
```

---

## Code

```java
import java.util.concurrent.*;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        int workers = 5;

        // Pattern 1: wait for workers to finish
        CountDownLatch done = new CountDownLatch(workers);
        ExecutorService pool = Executors.newFixedThreadPool(workers);

        for (int i = 0; i < workers; i++) {
            final int id = i;
            pool.submit(() -> {
                try {
                    Thread.sleep((long)(Math.random() * 1000));
                    System.out.println("Worker " + id + " done");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                } finally {
                    done.countDown(); // always countDown, even on exception
                }
            });
        }
        done.await(); // wait for all workers
        System.out.println("All workers finished!");

        // Pattern 2: start gun — all threads start together
        CountDownLatch startGun = new CountDownLatch(1);
        CountDownLatch allReady = new CountDownLatch(workers);

        for (int i = 0; i < workers; i++) {
            final int id = i;
            pool.submit(() -> {
                System.out.println("Worker " + id + " ready");
                allReady.countDown();
                try {
                    startGun.await(); // wait for start signal
                    System.out.println("Worker " + id + " STARTED!");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        allReady.await(); // wait until all are ready
        System.out.println("All ready — FIRE!");
        startGun.countDown(); // release all workers simultaneously

        pool.shutdown();
        pool.awaitTermination(10, TimeUnit.SECONDS);
    }
}
```

---

## Interview Questions

**Q: Can a `CountDownLatch` be reused?**
> No. Once the count reaches zero it stays at zero permanently. Use `CyclicBarrier` for reusable barriers.

**Q: What happens if `countDown()` is called more times than the initial count?**
> Nothing — `countDown()` is a no-op once the count is already 0.

**Q: What is the risk if `countDown()` is never called enough times?**
> Threads calling `await()` will block forever (deadlock). Always call `countDown()` in a `finally` block.

**Q: Can multiple threads call `await()`?**
> Yes — all waiting threads are released simultaneously when the count reaches zero.

---

## Cheat Sheet
| Method | Description |
|---|---|
| `new CountDownLatch(N)` | Initialize with count N |
| `countDown()` | Decrement count; releases if reaches 0 |
| `await()` | Block until count == 0 |
| `await(t, unit)` | Block with timeout |
| `getCount()` | Read current count |
| Reusable | ❌ No |
| Best for | One-time synchronization points |
