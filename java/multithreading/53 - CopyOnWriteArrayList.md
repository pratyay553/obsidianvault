# 53 - CopyOnWriteArrayList

Tags: #concurrent-collections #section9
Related: [[52 - ConcurrentHashMap]] | [[51 - Concurrent Collections]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand `CopyOnWriteArrayList` — a thread-safe list optimised for frequent reads and rare writes.

---

## Beginner Explanation
Every time you write to a `CopyOnWriteArrayList` (add, remove, set), it makes a **fresh copy of the entire underlying array**. Readers always see a stable snapshot — they never block and never throw `ConcurrentModificationException`.

---

## Analogy
It's like a notice board in an office. To post a new notice, you take the whole board down, make a photocopy with the new notice added, and put the new version up. Everyone reading the board sees a complete, consistent snapshot. Readers don't need to wait; writers just need to do extra copying work.

---

## Technical Deep Dive

### Internal mechanism
```
Write (add/remove/set):
  1. Acquire ReentrantLock
  2. Copy the backing array (O(n))
  3. Modify the copy
  4. Replace the volatile reference to point to the new array
  5. Release lock

Read (get/iterator/size):
  1. Read the volatile array reference → snapshot
  2. Access index directly — NO lock needed
```

### Trade-offs
| Aspect | Detail |
|---|---|
| Read performance | Excellent — lock-free, direct array access |
| Write performance | Poor for large lists — O(n) copy per write |
| Memory | High — keeps old array alive while readers hold snapshot |
| Iterators | Snapshot iterators — never throw CME, never see new writes |
| `remove(Object)` in iterator | Not supported — throws `UnsupportedOperationException` |

### When to use
✅ Read-heavy, write-rarely lists (event listener registries, config lists, allow/deny lists)
❌ Write-heavy lists (use `ConcurrentLinkedQueue` or `Collections.synchronizedList` instead)
❌ Large lists (each write copies the whole array)

### `CopyOnWriteArraySet`
Backed by a `CopyOnWriteArrayList`. Same trade-offs.

---

## Code

```java
import java.util.concurrent.*;
import java.util.*;

public class CopyOnWriteDemo {
    public static void main(String[] args) throws InterruptedException {
        CopyOnWriteArrayList<String> listeners = new CopyOnWriteArrayList<>();

        // Add listeners (rare)
        listeners.add("EmailNotifier");
        listeners.add("SlackNotifier");
        listeners.add("LogNotifier");

        // Many readers iterating — no locking, no CME
        ExecutorService readers = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            readers.submit(() -> {
                for (String listener : listeners) { // snapshot iteration
                    System.out.println(Thread.currentThread().getName() + " sees: " + listener);
                }
            });
        }

        // Writer modifying while readers iterate — safe, no CME
        listeners.add("SmsNotifier"); // creates new array copy
        listeners.remove("LogNotifier"); // creates another copy

        readers.shutdown();
        readers.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Final: " + listeners);

        // Iterator is a snapshot — won't see newly added items
        Iterator<String> it = listeners.iterator();
        listeners.add("PushNotifier");
        List<String> snapshot = new ArrayList<>();
        it.forEachRemaining(snapshot::add);
        System.out.println("Iterator snapshot: " + snapshot); // no PushNotifier
        System.out.println("Live list: " + listeners); // has PushNotifier
    }
}
```

---

## Interview Questions

**Q: How does `CopyOnWriteArrayList` achieve thread safety?**
> On every mutation (add, remove, set), it acquires a lock, copies the entire backing array, applies the change to the copy, then atomically replaces the array reference. Reads access the (volatile) array reference directly — no locking needed.

**Q: When should you use `CopyOnWriteArrayList` over `Collections.synchronizedList`?**
> When reads vastly outnumber writes and you can tolerate the memory and write overhead. `CopyOnWriteArrayList` gives lock-free reads; `synchronizedList` locks on every read AND write.

**Q: Does iterating over `CopyOnWriteArrayList` throw `ConcurrentModificationException`?**
> No. Iterators work on a snapshot of the array captured at iterator creation time. They're safe but may not reflect recent additions.

---

## Cheat Sheet
| Aspect | Detail |
|---|---|
| Write cost | O(n) — full array copy |
| Read cost | O(1) — no lock |
| Iterator | Snapshot — no CME, no live updates |
| Best for | Listener lists, config, allowlists |
| Avoid when | Write-heavy or large lists |
| Lock type | `ReentrantLock` on writes |
