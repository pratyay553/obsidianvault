# 55 - CyclicBarrier

Tags: #coordination-utilities #section9
Related: [[54 - CountDownLatch]] | [[56 - Phaser]] | [[57 - Semaphore]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `CyclicBarrier` — a reusable synchronization point where a fixed number of threads must all arrive before any of them continues.

---

## Beginner Explanation
A `CyclicBarrier` makes N threads wait for each other. When all N have called `await()`, they are all released simultaneously. Unlike `CountDownLatch`, it can be reused (reset automatically after each trip).

---

## Analogy
A group hiking trip: everyone must reach the checkpoint before the group moves on. Once the last hiker arrives at the checkpoint, the guide says "OK, everyone move!" and the cycle repeats at the next checkpoint.

---

## Technical Deep Dive

### API
```java
CyclicBarrier barrier = new CyclicBarrier(N);
CyclicBarrier barrier = new CyclicBarrier(N, barrierAction); // run when all arrive

barrier.await();              // block until N parties arrive
barrier.await(t, TimeUnit);   // with timeout
barrier.getNumberWaiting();   // how many currently waiting
barrier.getParties();         // N
barrier.reset();              // manually reset (breaks current waiters with BrokenBarrierException)
barrier.isBroken();           // true if barrier was broken
```

### How it works
1. Each thread calls `await()` and increments an internal counter
2. When the counter reaches N, the optional `barrierAction` Runnable runs (on the last-arriving thread)
3. All waiting threads are released; the barrier resets automatically for the next cycle

### BrokenBarrierException
Thrown when:
- A thread is interrupted while waiting at the barrier
- The barrier is `reset()` while threads are waiting
- Any waiting thread times out

Once broken, all threads waiting at the barrier receive `BrokenBarrierException` immediately.

### CountDownLatch vs CyclicBarrier

| Aspect | CountDownLatch | CyclicBarrier |
|---|---|---|
| Reusable | ❌ No | ✅ Yes |
| Who waits | One thread waits for many | All threads wait for each other |
| On completion | Nothing | Optional `Runnable` |
| Count type | Counts down | Counts up to N |

---

## Code

```java
import java.util.concurrent.*;

public class CyclicBarrierDemo {
    static final int THREADS = 3;
    static final int PHASES = 4; // simulate 4 rounds of parallel work

    public static void main(String[] args) throws InterruptedException {
        // Barrier action runs after all threads arrive each phase
        CyclicBarrier barrier = new CyclicBarrier(THREADS,
            () -> System.out.println(">>> Phase complete — all threads synchronized <<<\n")
        );

        ExecutorService pool = Executors.newFixedThreadPool(THREADS);
        for (int t = 0; t < THREADS; t++) {
            final int threadId = t;
            pool.submit(() -> {
                try {
                    for (int phase = 0; phase < PHASES; phase++) {
                        // Simulate varying amounts of work
                        long workMs = (long)(Math.random() * 500 + 100);
                        Thread.sleep(workMs);
                        System.out.printf("Thread %d finished phase %d (took %dms)%n",
                                          threadId, phase, workMs);
                        barrier.await(); // wait for all threads before next phase
                    }
                } catch (InterruptedException | BrokenBarrierException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        pool.shutdown();
        pool.awaitTermination(30, TimeUnit.SECONDS);

        // Example: parallel matrix computation using barrier
        System.out.println("\n--- Barrier reset demo ---");
        CyclicBarrier b2 = new CyclicBarrier(2);
        Thread t1 = new Thread(() -> {
            try {
                System.out.println("T1 phase 1 done"); b2.await();
                System.out.println("T1 phase 2 done"); b2.await(); // barrier reused!
            } catch (Exception e) { e.printStackTrace(); }
        });
        Thread t2 = new Thread(() -> {
            try {
                System.out.println("T2 phase 1 done"); b2.await();
                System.out.println("T2 phase 2 done"); b2.await();
            } catch (Exception e) { e.printStackTrace(); }
        });
        t1.start(); t2.start();
        t1.join(); t2.join();
    }
}
```

---

## Interview Questions

**Q: What is the difference between `CountDownLatch` and `CyclicBarrier`?**
> `CountDownLatch` is one-shot (non-reusable) and typically has one thread waiting for many workers to finish. `CyclicBarrier` is reusable — all N threads wait for each other, then proceed together through multiple phases.

**Q: What is `BrokenBarrierException`?**
> Thrown when a thread tries to wait at a broken barrier. A barrier breaks when a waiting thread is interrupted, times out, or `reset()` is called while threads are waiting.

**Q: What does the optional `Runnable` in `CyclicBarrier` do?**
> It runs once when the last thread arrives at the barrier, before any threads are released. Useful for phase-transition logic (aggregating results, printing phase summary, etc.).

---

## Cheat Sheet
| Method | Description |
|---|---|
| `new CyclicBarrier(N)` | N threads must meet |
| `new CyclicBarrier(N, action)` | Run action when all N arrive |
| `await()` | Block until all N arrive |
| `reset()` | Manually reset (breaks current waiters) |
| `isBroken()` | Check if barrier was broken |
| Reusable | ✅ Yes — resets automatically |
