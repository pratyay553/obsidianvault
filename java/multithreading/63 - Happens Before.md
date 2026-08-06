# 63 - Happens Before

Tags: #java-memory-model #section11
Related: [[62 - Java Memory Model]] | [[29 - Volatile]] | [[22 - Synchronization]] | [[64 - Memory Consistency]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand the **happens-before** (HB) relation — the core rule of the Java Memory Model that defines when a write is guaranteed to be visible to a read.

---

## Beginner Explanation
"A happens-before B" means: everything A did (writes to variables) is **guaranteed to be visible** to B before B executes. If there's no HB relation between a write and a read, the read may see stale data — even if the write happened "earlier" in wall-clock time.

---

## Analogy
Sending a certified letter. If you send certified mail (HB), the recipient is guaranteed to receive your message before they reply. If you shout across a noisy room (no HB), you can't guarantee they heard you at all.

---

## Technical Deep Dive

### HB rules (from JLS §17.4.5)

| Rule | Description |
|---|---|
| **Program Order** | Within a single thread, each action HB every subsequent action in that thread |
| **Monitor Lock** | An `unlock` of monitor M HB before every subsequent `lock` of M |
| **Volatile Write** | A write to volatile field v HB before every subsequent read of v |
| **Thread Start** | `Thread.start()` HB before any action in the started thread |
| **Thread Join** | Any action in thread T HB before `Thread.join()` returns in another thread |
| **Object Init** | Default initialization of fields HB before the constructor starts |
| **Transitivity** | If X HB Y and Y HB Z, then X HB Z |

### What HB gives you
1. **Visibility**: the reader sees all writes by the writer that HB the read
2. **Ordering**: actions before the HB point appear ordered before actions after it

### What HB does NOT give you
- HB is not total ordering — two unrelated actions have no HB relation
- HB is not the same as "executed before in wall-clock time"
- Lack of HB = **data race** = undefined behavior (in JMM terms)

### Common pitfall: flag without volatile

```java
boolean done = false;   // no volatile — no HB between threads
// Thread A:
doWork();
done = true;            // write — no HB to Thread B's read
// Thread B:
while (!done) {}        // read — may never see done = true
```

Fix: `volatile boolean done` — the volatile write HB the volatile read.

### How `synchronized` establishes HB

```
Thread A:                       Thread B:
synchronized(lock) {            synchronized(lock) {
    x = 1;                 HB       y = x;  // guaranteed to see x=1
}                               }
```
Unlock HB lock → transitivity → all writes before unlock HB all reads after lock.

---

## Code

```java
public class HappensBeforeDemo {
    static int data = 0;
    static volatile boolean ready = false;

    public static void main(String[] args) throws InterruptedException {
        Thread writer = new Thread(() -> {
            data  = 42;       // regular write
            ready = true;     // volatile write — HB fence
        });

        Thread reader = new Thread(() -> {
            while (!ready) {} // volatile read — sees the volatile write
            // Transitivity: volatile write HB volatile read
            // data=42 HB volatile write HB volatile read HB this line
            System.out.println("data = " + data); // guaranteed to see 42
        });

        writer.start();
        reader.start();
        writer.join();
        reader.join();
    }
}
```

The key: `data = 42` HB `ready = true` (program order) HB `while (!ready)` returns (volatile read). By transitivity, the reader sees `data = 42`.

---

## Interview Questions

**Q: What does "happens-before" mean in Java?**
> If action A happens-before action B, then all memory writes made by A (and anything A can see) are guaranteed to be visible to B. It's the JMM's formal mechanism for reasoning about visibility and ordering.

**Q: Does `volatile` establish a happens-before relation?**
> Yes. A write to a `volatile` field happens-before every subsequent read of that same field. This makes the volatile write and all prior writes visible to the reader.

**Q: If Thread A writes x=1 before Thread B reads x, does B always see 1?**
> Only if there is a happens-before relation between the write and the read. If they're both unsynchronized, B may see stale data — wall-clock order is not enough.

**Q: What is a data race?**
> When two threads access the same variable, at least one access is a write, and there is no happens-before relation between them. Data races cause undefined/unpredictable behavior under the JMM.

---

## Cheat Sheet
| HB Rule | How to establish |
|---|---|
| Between threads | `volatile`, `synchronized`, locks, `Thread.start()`/`join()` |
| Within a thread | Program order (free) |
| Transitivity | Chain HB rules |
| No HB + shared mutable = | Data race → undefined behavior |
