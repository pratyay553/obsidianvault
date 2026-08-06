# 66 - CPU Cache

Tags: #java-memory-model #section11 #hardware
Related: [[65 - False Sharing]] | [[62 - Java Memory Model]] | [[72 - Performance Tuning]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand CPU cache architecture and why it matters for writing high-performance concurrent Java code.

---

## Beginner Explanation
CPUs are much faster than RAM. To bridge this gap, CPUs have small, fast caches (L1, L2, L3) that store recently used data. Understanding these caches explains why multithreaded bugs and performance problems occur.

---

## Analogy
A chef's workstation:
- **L1 cache** — ingredients on the cutting board (tiny, instant access)
- **L2 cache** — ingredients in the fridge next to the stove (small, fast)
- **L3 cache** — the walk-in cooler shared by all chefs (bigger, slower)
- **Main memory (RAM)** — the supplier warehouse across the street (huge, slow)
- **Disk** — a distant farm (enormous, very slow)

---

## Technical Deep Dive

### Cache hierarchy (typical modern CPU)

| Level | Size | Latency | Shared? |
|---|---|---|---|
| L1 (I+D) | 32–64 KB | ~4 cycles (~1 ns) | Per core |
| L2 | 256 KB – 1 MB | ~12 cycles (~3 ns) | Per core |
| L3 | 4–64 MB | ~40 cycles (~10 ns) | Shared across all cores |
| RAM | GBs | ~100–200 cycles (~50 ns) | System-wide |

### Cache line
The unit of movement between cache and RAM is a **cache line** — 64 bytes on x86. An entire 64-byte block is loaded even if you access 1 byte. This is why adjacent array elements are fast (spatial locality) but random access is slow.

### Cache coherence (MESI protocol)
When one core writes to a cache line, the MESI protocol ensures other cores' copies are invalidated:
```
Core A writes x → broadcasts INVALIDATE to all cores
Core B must reload the cache line before it can read x
```
This is what makes `volatile` work — it inserts memory barriers that force cache flush/invalidation.

### Implications for Java code

**Spatial locality** — access array elements sequentially (row-major for 2D arrays):
```java
// FAST — sequential cache line access
for (int i = 0; i < N; i++) sum += arr[i];

// SLOW — random/strided access (cache misses)
for (int i = 0; i < N; i += 64) sum += arr[i];
```

**False sharing** — two threads writing to different variables on the same cache line → [[65 - False Sharing]]

**Object layout** — Java objects are laid out in memory by the JVM. Closely related fields tend to be on the same cache line (good for single-threaded access, bad for multi-threaded writes to those fields).

### JVM tools for cache analysis
- `-XX:+PrintCompilation -XX:+PrintInlining` — JIT decisions
- `jstat` — GC and memory stats
- Java Flight Recorder (JFR) — hardware counter events (`CacheMisses`)
- `perf stat -e cache-misses java MyApp` (Linux)

---

## Code

```java
public class CacheFriendlyDemo {
    static final int SIZE = 1024;
    static int[][] matrix = new int[SIZE][SIZE];

    // Cache-friendly: row-major traversal (sequential cache line access)
    static long rowMajorSum() {
        long sum = 0;
        for (int i = 0; i < SIZE; i++)
            for (int j = 0; j < SIZE; j++)
                sum += matrix[i][j]; // sequential — good spatial locality
        return sum;
    }

    // Cache-unfriendly: column-major traversal (strided — many cache misses)
    static long colMajorSum() {
        long sum = 0;
        for (int j = 0; j < SIZE; j++)
            for (int i = 0; i < SIZE; i++)
                sum += matrix[i][j]; // stride=SIZE — cache miss every access
        return sum;
    }

    public static void main(String[] args) {
        // Warm up
        rowMajorSum(); colMajorSum();

        long t1 = System.nanoTime();
        rowMajorSum();
        System.out.println("Row-major: " + (System.nanoTime() - t1) / 1_000_000 + " ms");

        long t2 = System.nanoTime();
        colMajorSum();
        System.out.println("Col-major: " + (System.nanoTime() - t2) / 1_000_000 + " ms");
        // Row-major is typically 3–10x faster due to cache friendliness
    }
}
```

---

## Interview Questions

**Q: What is a CPU cache line and why does it matter for Java?**
> A 64-byte unit of cache movement. Memory is loaded/invalidated at cache-line granularity. This causes false sharing (two threads writing to different variables on the same cache line fight for ownership) and affects spatial locality performance.

**Q: Why is sequential array access faster than random access in Java?**
> Sequential access exploits spatial locality — adjacent elements are on the same cache line. Loading one element prefetches its neighbors. Random access causes a cache miss for every element.

**Q: How does `volatile` interact with CPU caches?**
> A `volatile` write inserts a store-barrier that flushes the CPU's write buffer and marks other cores' copies of the cache line as invalid. A `volatile` read inserts a load-barrier that forces a fresh load from the coherent cache.

---

## Cheat Sheet
| Level | Latency | Scope |
|---|---|---|
| L1 | ~1 ns | Per core |
| L2 | ~3 ns | Per core |
| L3 | ~10 ns | Shared |
| RAM | ~50 ns | System |
| Cache line | 64 bytes | Unit of movement |
| False sharing | Two vars in same line | Fix: padding / `@Contended` |
| Spatial locality | Sequential > random | Affects hot loops |
