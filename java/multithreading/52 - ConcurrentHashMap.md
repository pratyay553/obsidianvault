# 52 - ConcurrentHashMap

Tags: #concurrent-collections #section9
Related: [[51 - Concurrent Collections]] | [[53 - CopyOnWriteArrayList]] | [[22 - Synchronization]] | [[29 - Volatile]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master `ConcurrentHashMap` — the thread-safe, high-performance alternative to `HashMap` and `Hashtable` for concurrent use.

---

## Beginner Explanation
A regular `HashMap` is not thread-safe. `Hashtable` is safe but locks the whole map on every operation (slow). `ConcurrentHashMap` is thread-safe **and** fast, because it only locks small segments of the map rather than the whole thing.

---

## Analogy
Think of a supermarket. `Hashtable` has one checkout lane that everyone must queue for. `ConcurrentHashMap` has 16+ lanes open simultaneously — customers (threads) mostly don't block each other.

---

## Technical Deep Dive

### Internal structure (Java 8+)
- Array of nodes (bins), each bin is a linked list or red-black tree when it grows long
- **No segment-level locking** (Java 8+ removed segments from Java 6/7)
- Uses **CAS** (Compare-And-Swap) for most operations → [[31 - CAS]]
- Only locks individual **bin heads** when modifying a bin (fine-grained `synchronized`)
- Reads are almost always **lock-free** thanks to `volatile` node references

### Concurrency level
Default concurrency: **16 simultaneous writers** at different bins can proceed without blocking each other.

### Key behaviours

| Operation | Thread Safety | How |
|---|---|---|
| `get(key)` | Lock-free | `volatile` reads |
| `put(key, val)` | Lock-free (empty bin) or bin-level lock | CAS or `synchronized(bin)` |
| `remove(key)` | Bin-level lock | `synchronized(bin)` |
| `size()` | Approximate | Summed `CounterCell` array |
| `putIfAbsent` | Atomic | CAS or bin lock |
| `computeIfAbsent` | Atomic | Bin-level lock |
| `compute` | Atomic | Bin-level lock |
| `merge` | Atomic | Bin-level lock |

### Important: `size()` is approximate
Under concurrent modification, `size()` may not be exactly accurate. Use `mappingCount()` (returns `long`) for large maps.

### `computeIfAbsent` — the key pattern
```java
map.computeIfAbsent(key, k -> expensiveCompute(k))
```
Atomically: check if key exists → if not, compute and insert. No external synchronization needed.

### What `ConcurrentHashMap` does NOT support
- `null` keys or values (throws `NullPointerException`)
- Atomic compound operations spanning multiple keys without external locks
- Consistent snapshots of the entire map (iteration is weakly consistent)

### Weakly consistent iterators
Iterators reflect the map as it was at or after their creation. They do not throw `ConcurrentModificationException` and may or may not reflect later updates.

---

## Code

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

public class ConcurrentHashMapDemo {
    public static void main(String[] args) throws InterruptedException {
        ConcurrentHashMap<String, AtomicInteger> wordCount = new ConcurrentHashMap<>();

        // Safe concurrent word counting
        String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
        ExecutorService pool = Executors.newFixedThreadPool(4);

        for (String word : words) {
            pool.submit(() ->
                wordCount.computeIfAbsent(word, k -> new AtomicInteger(0))
                         .incrementAndGet()
            );
        }
        pool.shutdown();
        pool.awaitTermination(5, TimeUnit.SECONDS);

        wordCount.forEach((word, count) ->
            System.out.println(word + ": " + count.get())
        );

        // putIfAbsent — atomic check-then-insert
        ConcurrentHashMap<String, String> cache = new ConcurrentHashMap<>();
        String prev = cache.putIfAbsent("key", "value1");
        System.out.println("Was null: " + (prev == null)); // true — inserted
        cache.putIfAbsent("key", "value2"); // ignored — key exists
        System.out.println(cache.get("key")); // value1

        // merge — atomic update
        ConcurrentHashMap<String, Integer> scores = new ConcurrentHashMap<>();
        scores.merge("player1", 10, Integer::sum); // 10
        scores.merge("player1", 5, Integer::sum);  // 15
        System.out.println("Score: " + scores.get("player1")); // 15

        // Bulk operations (Java 8+)
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        map.put("a", 1); map.put("b", 2); map.put("c", 3);

        // forEach with parallelism threshold
        map.forEach(1, (k, v) -> System.out.println(k + "=" + v));

        // reduce
        int sum = map.reduceValues(1, Integer::sum);
        System.out.println("Sum: " + sum); // 6

        // search — returns first non-null result
        String found = map.search(1, (k, v) -> v > 1 ? k : null);
        System.out.println("Found key with v>1: " + found);
    }
}
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `if (!map.containsKey(k)) map.put(k, v)` | Use `putIfAbsent` or `computeIfAbsent` |
| Using `null` key or value | Not allowed; use sentinel values |
| Treating `size()` as exact | Use `mappingCount()` for large maps |
| Using iterators as consistent snapshots | Iterators are weakly consistent |
| `get` then `put` as one atomic operation | Use `compute` or `merge` instead |

---

## Interview Questions

**Q: How does `ConcurrentHashMap` achieve thread safety without locking the whole map?**
> Java 8+ uses CAS operations and per-bin `synchronized` blocks. Reads are lock-free via `volatile` node references. Writes only lock the specific bin being modified, so 16+ concurrent writers can proceed simultaneously.

**Q: Why does `ConcurrentHashMap` not allow null keys or values?**
> In a concurrent context, `map.get(key) == null` is ambiguous: it could mean the key doesn't exist, or it maps to null. `ConcurrentHashMap` eliminates this ambiguity by banning null.

**Q: What is the difference between `put` and `putIfAbsent`?**
> `put` always overwrites. `putIfAbsent` is atomic: it only inserts if the key is absent and returns the existing value if present.

**Q: What is a weakly consistent iterator?**
> It reflects the map state at or after iterator creation. It will not throw `ConcurrentModificationException`, but may or may not show updates made after the iterator was created.

**Q: How is `ConcurrentHashMap` different from `Collections.synchronizedMap`?**
> `synchronizedMap` wraps every method with a single lock — one thread at a time. `ConcurrentHashMap` allows multiple concurrent readers and writers (at different bins) simultaneously.

---

## Cheat Sheet

| Method            | Atomicity             |
| ----------------- | --------------------- |
| `get`             | Lock-free             |
| `put`             | Bin-level             |
| `putIfAbsent`     | Atomic                |
| `computeIfAbsent` | Atomic                |
| `compute`         | Atomic                |
| `merge`           | Atomic                |
| `size()`          | Approximate           |
| `mappingCount()`  | Better for large maps |
| Null keys/values  | ❌ Not allowed         |
| Iterators         | Weakly consistent     |
