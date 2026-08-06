# 65 - False Sharing

Tags: #java-memory-model #section11 #performance
Related: [[66 - CPU Cache]] | [[62 - Java Memory Model]] | [[72 - Performance Tuning]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand false sharing — a silent performance killer where threads on different cores unknowingly compete for the same CPU cache line.

---

## Beginner Explanation
CPUs move memory in chunks called **cache lines** (typically 64 bytes). If two variables happen to sit in the same cache line, and two different threads write to those variables simultaneously, the cache line gets invalidated and reloaded on every write — even though the threads are writing to **different** variables.

---

## Analogy
Two chefs sharing a small tray (cache line). Chef A only uses the left side of the tray (variable X), Chef B only uses the right side (variable Y). But every time either chef touches the tray, the whole tray has to be passed between them, even though they're never actually sharing food.

---

## Technical Deep Dive

### Cache coherence protocol (MESI)
Modern CPUs use the MESI protocol (Modified, Exclusive, Shared, Invalid) to keep caches consistent. When one core writes to a cache line, it broadcasts an **invalidation** signal to all other cores holding that line. They must reload the line from shared (L3) cache or main memory.

### False sharing scenario
```
Cache line (64 bytes):
[ counter1 (8 bytes) | counter2 (8 bytes) | ... padding ... ]

Thread 1 on Core 1: counter1++ → invalidates the entire cache line on Core 2
Thread 2 on Core 2: counter2++ → invalidates the entire cache line on Core 1
→ Both cores constantly wait for the cache line — performance collapses
```

### How to detect false sharing
- **Java Flight Recorder** + JFR events for cache misses
- **Linux perf**: `perf stat -e cache-misses,cache-references ./app`
- **Intel VTune**: hardware counter analysis
- Look for: throughput drops under concurrency with no apparent lock contention

### Solutions

**1. Padding** — manually add unused bytes to push variables to separate cache lines
```java
// BEFORE (false sharing):
long counter1, counter2; // adjacent — may share a cache line

// AFTER (padded):
long counter1;
long p1, p2, p3, p4, p5, p6, p7; // 7 longs = 56 bytes padding
long counter2; // now on a different cache line
```

**2. `@Contended` annotation (Java 8+)** — JVM pads the field automatically
```java
import sun.misc.Contended; // internal; use carefully
@Contended
long counter1;
@Contended
long counter2;
```
Requires JVM flag: `-XX:-RestrictContended`

**3. Thread-local accumulation** — each thread keeps its own counter and merges at the end (most efficient for counters)
```java
// Each thread works on its own variable, no sharing at all
LongAdder adder = new LongAdder(); // internally uses Striped64 — avoids false sharing
adder.increment(); // thread-local cell
long total = adder.sum(); // merge at the end
```

---

## Code

```java
import java.util.concurrent.atomic.*;

public class FalseSharingDemo {
    static final int THREADS = 4;
    static final int ITERATIONS = 100_000_000;

    // BAD — potential false sharing (two longs in adjacent memory)
    static long[] badCounters = new long[THREADS];

    // GOOD — padded to separate cache lines
    static long[][] goodCounters = new long[THREADS][8]; // 8 longs = 64 bytes per slot

    // BEST — use LongAdder (Striped64 avoids false sharing internally)
    static LongAdder bestCounter = new LongAdder();

    static void benchmark(String label, Runnable setup, Runnable work) throws InterruptedException {
        setup.run();
        Thread[] threads = new Thread[THREADS];
        long start = System.currentTimeMillis();
        for (int t = 0; t < THREADS; t++) { threads[t] = new Thread(work); threads[t].start(); }
        for (Thread t : threads) t.join();
        System.out.printf("%-20s: %d ms%n", label, System.currentTimeMillis() - start);
    }

    public static void main(String[] args) throws InterruptedException {
        benchmark("False sharing", () -> {},
            () -> { int id = (int)(Thread.currentThread().getId() % THREADS);
                    for (int i = 0; i < ITERATIONS / THREADS; i++) badCounters[id]++; });

        benchmark("Padded", () -> {},
            () -> { int id = (int)(Thread.currentThread().getId() % THREADS);
                    for (int i = 0; i < ITERATIONS / THREADS; i++) goodCounters[id][0]++; });

        benchmark("LongAdder", () -> {},
            () -> { for (int i = 0; i < ITERATIONS / THREADS; i++) bestCounter.increment(); });
    }
}
```

---

## Interview Questions

**Q: What is false sharing?**
> Two threads write to different variables that happen to reside in the same CPU cache line. Every write by either thread invalidates the cache line in the other core's cache, causing constant cache misses and drastically reducing throughput.

**Q: How do you fix false sharing?**
> 1. Pad with unused bytes between variables to push them to separate cache lines. 2. Use `@Contended` annotation (Java 8+). 3. Use thread-local accumulators (`LongAdder`). 4. Redesign to avoid sharing.

**Q: Why does `LongAdder` outperform `AtomicLong` under contention?**
> `LongAdder` uses `Striped64` — an array of cells, one per contending thread (padded to avoid false sharing). Each thread updates its own cell. `sum()` aggregates at the end. No false sharing, no CAS retry loops.

---

## Cheat Sheet
| Concept | Detail |
|---|---|
| Cache line | 64 bytes — unit of CPU cache movement |
| False sharing | Different vars in same cache line cause ping-pong |
| Detection | `perf stat`, JFR, VTune |
| Fix 1 | Manual padding (7 longs = 56 bytes) |
| Fix 2 | `@Contended` annotation + JVM flag |
| Fix 3 | `LongAdder` / thread-local accumulation |
