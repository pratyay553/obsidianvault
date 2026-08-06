# 72 - Performance Tuning

Tags: #jvm-internals #section12 #performance
Related: [[71 - Profiling Threads]] | [[44 - Thread Pool]] | [[65 - False Sharing]] | [[12 - Context Switching]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Learn how to tune thread pool sizes, reduce lock contention, and optimize concurrent Java code for production throughput and latency.

---

## Beginner Explanation
More threads is not always better. The right number depends on whether your work is CPU-bound or I/O-bound. Beyond a certain point, adding threads increases context switching and memory overhead, hurting performance.

---

## Technical Deep Dive

### Rule 1 — Thread pool sizing formula

**CPU-bound** (compression, encryption, computation):
```
Optimal threads ≈ Number of CPU cores
```
Adding more threads doesn't help — they fight for the same CPUs and cause context switches.

**I/O-bound** (DB calls, HTTP calls, file I/O):
```
Optimal threads ≈ cores × (1 + wait_time / service_time)
```
If threads spend 90% of time waiting (wait_time / service_time = 9), optimal = cores × 10.

```java
int cores = Runtime.getRuntime().availableProcessors();
// CPU-bound:
int cpuPoolSize = cores;
// I/O-bound (assuming 9:1 wait:work ratio):
int ioPoolSize = cores * (1 + 9); // = cores * 10
```

### Rule 2 — Use the right pool type

| Pool | Use case |
|---|---|
| `newFixedThreadPool(N)` | Known I/O-bound load with known N |
| `newCachedThreadPool()` | Short-lived tasks with variable load (careful: unbounded) |
| `ForkJoinPool.commonPool()` | CPU-bound parallel computation |
| Virtual threads (Java 21+) | High-concurrency I/O — one-thread-per-task is OK |

### Rule 3 — Reduce lock contention

1. **Minimize lock scope** — hold locks for the shortest time possible
2. **Use `ReadWriteLock`** for read-heavy access → [[33 - ReadWriteLock]]
3. **Prefer atomics over locks** for single-variable operations → [[30 - Atomic Variables]]
4. **Use `ConcurrentHashMap`** instead of synchronized maps → [[52 - ConcurrentHashMap]]
5. **Use `LongAdder`** instead of `AtomicLong` for high-contention counters
6. **Lock striping** — use multiple locks, each protecting a subset of data

### Rule 4 — Avoid false sharing → [[65 - False Sharing]]
Use `@Contended`, padding, or `LongAdder` for per-thread counters.

### Rule 5 — Reduce object allocation in hot paths
Allocation → GC pressure → STW pauses → latency spikes.
- Use object pools for expensive objects (DB connections, buffers)
- Prefer primitives over boxed types in hot loops
- Use `LongAdder` / `DoubleAdder` — they use striped cells (less GC pressure)

### Rule 6 — Measure before optimizing
```bash
# JVM flags for performance analysis
-XX:+PrintGCDetails -XX:+PrintGCDateStamps   # GC analysis
-XX:+PrintCompilation                         # JIT decisions
-Xss256k                                      # reduce stack size per thread
-XX:+UseZGC                                   # low-latency GC (Java 15+)
```

### Key metrics to track
| Metric | Tool | Target |
|---|---|---|
| Thread count | `tmx.getThreadCount()` | < 500 (platform threads) |
| Lock wait time | JFR `JavaMonitorEnter` | < 1ms p99 |
| Context switches | `vmstat`, `pidstat -w` | < 10k/sec |
| GC pause p99 | JFR, GC logs | < 100ms (G1), < 5ms (ZGC) |
| CPU utilization | `top`, JFR | < 80% sustained |

---

## Code

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

public class PerformanceTuningDemo {
    static final int CORES = Runtime.getRuntime().availableProcessors();

    // BAD: AtomicLong under high contention — CAS retry loop
    static AtomicLong atomicCounter = new AtomicLong(0);

    // GOOD: LongAdder — striped cells, no contention
    static LongAdder adderCounter = new LongAdder();

    public static void main(String[] args) throws InterruptedException {
        int threads = CORES * 4;
        int iters = 10_000_000 / threads;

        // Benchmark AtomicLong
        long t1 = System.currentTimeMillis();
        runBenchmark(threads, iters, () -> atomicCounter.incrementAndGet());
        System.out.println("AtomicLong: " + (System.currentTimeMillis() - t1) + "ms");

        // Benchmark LongAdder
        long t2 = System.currentTimeMillis();
        runBenchmark(threads, iters, () -> adderCounter.increment());
        System.out.println("LongAdder:  " + (System.currentTimeMillis() - t2) + "ms");
        // LongAdder is typically 3-10x faster under high contention

        System.out.println("Cores: " + CORES);
        System.out.printf("CPU-bound pool size: %d%n", CORES);
        System.out.printf("I/O-bound pool size (9:1 ratio): %d%n", CORES * 10);
    }

    static void runBenchmark(int threads, int iters, Runnable op) throws InterruptedException {
        ExecutorService pool = Executors.newFixedThreadPool(threads);
        CountDownLatch latch = new CountDownLatch(threads);
        for (int t = 0; t < threads; t++) {
            pool.submit(() -> {
                for (int i = 0; i < iters; i++) op.run();
                latch.countDown();
            });
        }
        latch.await();
        pool.shutdown();
    }
}
```

---

## Interview Questions

**Q: How do you determine the optimal thread pool size?**
> For CPU-bound: N ≈ number of CPU cores. For I/O-bound: N ≈ cores × (1 + wait_time/service_time). Always measure under realistic load — theory is a starting point.

**Q: Why is `LongAdder` faster than `AtomicLong` under contention?**
> `AtomicLong` uses a single CAS loop — under contention, threads retry. `LongAdder` uses a striped array of cells (one per contending thread), each padded to avoid false sharing. No retries; aggregate with `sum()` at the end.

**Q: What is lock striping?**
> Splitting a lock into N locks, each protecting a subset of data. `ConcurrentHashMap` does this internally per bin. Reduces contention by a factor of N.

---

## Cheat Sheet
| Rule | Detail |
|---|---|
| CPU-bound pool | N = CPU cores |
| I/O-bound pool | N = cores × (1 + wait/service) |
| Counter under contention | `LongAdder` > `AtomicLong` |
| Map | `ConcurrentHashMap` > `synchronizedMap` |
| Lock scope | Minimize time held |
| False sharing | Padding / `@Contended` / `LongAdder` |
| GC | ZGC for low-latency; fix heap size with -Xms = -Xmx |
