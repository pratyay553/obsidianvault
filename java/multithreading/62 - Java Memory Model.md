# 62 - Java Memory Model

Tags: #java-memory-model #section11
Related: [[63 - Happens Before]] | [[64 - Memory Consistency]] | [[29 - Volatile]] | [[22 - Synchronization]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand the Java Memory Model (JMM) — the rules that define how threads see each other's memory operations and when changes become visible.

---

## Beginner Explanation
Modern CPUs and compilers optimize code by reordering instructions and caching values in registers/CPU caches. Without rules, Thread A might not see Thread B's writes at all. The JMM defines the **guarantee**: under what conditions a write by one thread is visible to reads by another.

---

## Analogy
Imagine two employees working in separate offices (CPU caches). They each write notes (variables) on their own whiteboards. Without a rule, they might never share those notes with each other. The JMM is the rule that says "after you synchronize, your whiteboard gets flushed to the shared noticeboard, and the other person's copy is refreshed."

---

## Technical Deep Dive

### The memory visibility problem

Without the JMM guarantees, this code is broken:
```java
boolean stop = false;     // shared variable — no volatile

// Thread A:
while (!stop) { }

// Thread B:
stop = true; // Thread A may NEVER see this!
```
The CPU may cache `stop` in Thread A's register and never re-read it from main memory.

### JMM key concepts

**Main memory vs working memory**
- Main memory: the shared heap
- Working memory: per-thread cache (CPU registers + L1/L2 cache)
- Without synchronization, writes to main memory may be delayed, and reads may come from stale working memory

**Happens-Before (HB) relation** → [[63 - Happens Before]]
The JMM guarantees visibility through HB relationships. If action X happens-before action Y, then Y sees the effects of X. Key HB rules:

| HB Rule | Description |
|---|---|
| Program order | Within one thread, statements execute in order |
| Monitor lock | Unlock of a lock HB before subsequent lock of same lock |
| Volatile write | Write to volatile field HB before subsequent read of same field |
| Thread start | `t.start()` HB before any action in thread t |
| Thread join | All actions in thread t HB before `t.join()` returns |
| Transitivity | If X HB Y and Y HB Z, then X HB Z |

### What the JMM does NOT guarantee without synchronization
- **Visibility**: Thread A's write may not be visible to Thread B
- **Atomicity**: 64-bit reads/writes (`long`, `double`) may be split into two 32-bit operations on some platforms
- **Ordering**: compiler/CPU may reorder instructions within a thread (as long as single-thread semantics are preserved)

### Synchronization mechanisms that establish HB
- `synchronized` blocks → [[22 - Synchronization]]
- `volatile` → [[29 - Volatile]]
- `java.util.concurrent` locks (`ReentrantLock`, etc.) → [[32 - ReentrantLock]]
- `Thread.start()` / `Thread.join()`
- `CountDownLatch`, `Semaphore`, etc.

---

## Code

```java
public class JMMDemo {
    // Without volatile — broken
    static boolean ready = false;
    static int value = 0;

    // With volatile — correct
    static volatile boolean readyV = false;
    static volatile int valueV = 0;

    public static void main(String[] args) throws InterruptedException {
        // BROKEN: no HB guarantee
        Thread writer = new Thread(() -> {
            value = 42;
            ready = true;  // may not be visible to reader
        });
        Thread reader = new Thread(() -> {
            while (!ready) {}  // may loop forever OR see ready=true but value=0
            System.out.println("Value: " + value);
        });
        writer.start(); reader.start();
        writer.join(); reader.join();

        // CORRECT: volatile establishes HB
        Thread w2 = new Thread(() -> {
            valueV = 42;
            readyV = true;  // volatile write — HB before next volatile read
        });
        Thread r2 = new Thread(() -> {
            while (!readyV) {}  // volatile read — sees the write
            System.out.println("ValueV: " + valueV);  // guaranteed to see 42
        });
        w2.start(); r2.start();
        w2.join(); r2.join();
    }
}
```

---

## Interview Questions

**Q: What is the Java Memory Model?**
> A specification (part of the Java Language Specification) that defines the rules for when writes by one thread become visible to reads by another. It uses the happens-before relation to reason about visibility and ordering.

**Q: What is the visibility problem in multithreading?**
> Without synchronization, CPU caches and compiler optimizations mean one thread's write to a shared variable may never be seen by another thread. The JMM guarantees visibility only through established happens-before relationships.

**Q: What guarantees does the JMM make about `long` and `double`?**
> Without `volatile`, reads and writes of `long` and `double` (64-bit) are not guaranteed to be atomic — they can be split into two 32-bit operations. Declaring them `volatile` makes them atomic.

---

## Cheat Sheet
| Issue | JMM cause | Fix |
|---|---|---|
| Stale read | No HB relation | `volatile`, `synchronized`, locks |
| Reordering | CPU/compiler optimization | `volatile`, memory barriers |
| 64-bit non-atomicity | Split operations | `volatile long/double` |
| Key mechanism | Happens-Before relation | See [[63 - Happens Before]] |
