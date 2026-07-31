# Monitor Lock

## Objective

By the end of this note, you will understand:

- What a monitor is, precisely, in the JVM's threading model
- How every Java object inherently has an associated monitor lock
- How the monitor relates to `synchronized`, `wait()`, and `notify()`
- What actually happens at the JVM/bytecode level when you enter a `synchronized` block

This note builds directly on [[22 - Synchronization]] and is the conceptual foundation for [[24 - Intrinsic Lock]] and [[25 - synchronized Keyword]].

---

## Why should I learn this?

"Monitor" is a term you'll see constantly in Java concurrency documentation, thread dumps, and interview questions — phrases like "waiting to lock monitor" or "acquiring the object's monitor" appear everywhere once you start looking. If you don't know precisely what a monitor **is**, these phrases remain vague jargon. Once you understand it, `synchronized`, `wait()`, and `notify()` all click together as different operations on the **same underlying object**: the monitor.

---

## Beginner Explanation

Every single object you create in Java — literally every object, not just special ones — comes with an invisible, built-in "lock" attached to it, called its **monitor**. Think of it like every object secretly having its own small padlock built in, even though you never explicitly created that padlock yourself.

When code says `synchronized(someObject)`, it's really saying: **"try to grab `someObject`'s built-in padlock before continuing; if someone else already has it, wait your turn."** Only one thread can hold a given object's monitor lock at any one time.

---

## Real World Analogy

**Hospital Analogy**

Imagine every patient room in a hospital has a "do not disturb" sign that only one nurse can flip to "in use" at a time. When Nurse A enters a room to do a procedure, they flip the sign to "in use" (acquire the monitor). If Nurse B tries to enter the same room while the sign is flipped, they must wait outside until Nurse A finishes and flips the sign back (releases the monitor).

Crucially, **every room** has its own independent sign — a nurse working in Room 101 doesn't block anyone from entering Room 102. Similarly, **every Java object** has its own independent monitor — locking on `objectA` has no effect on `objectB`'s monitor whatsoever.

---

## Technical Explanation

### Every Object Has a Monitor

In the JVM, **every object** (an instance of any class) has an associated monitor, sometimes also called its "intrinsic lock" (see [[24 - Intrinsic Lock]] for more on this exact terminology). This isn't something you opt into — it's a built-in structural part of every object header in the JVM's internal memory representation.

### What a Monitor Provides

A monitor provides two related capabilities:

1. **Mutual exclusion**: Only one thread can "own" (hold) a given object's monitor at a time.
2. **Wait/notify coordination**: A thread that currently owns a monitor can voluntarily give it up temporarily via `wait()`, and be woken up later via `notify()`/`notifyAll()` called by another thread that acquires the same monitor (fully detailed in [[40 - wait()]], [[41 - notify()]], [[42 - notifyAll()]]).

### How `synchronized` Uses the Monitor

```java
Object lock = new Object();

synchronized (lock) {
    // Only one thread can be inside this block for THIS specific 'lock' object at a time
}
```

When a thread reaches `synchronized (lock) { ... }`:
1. It attempts to **acquire** `lock`'s monitor.
2. If no other thread currently holds it, the thread acquires it immediately and proceeds.
3. If another thread already holds it, this thread enters the `BLOCKED` state (see [[09 - Thread Lifecycle]]) until the monitor becomes available.
4. Upon exiting the `synchronized` block (normally or via an exception), the thread automatically **releases** the monitor.

### Synchronized Instance Methods and the Monitor

```java
class Counter {
    public synchronized void increment() { /* ... */ }
}
```

A `synchronized` **instance method** is equivalent to wrapping the entire method body in `synchronized(this) { ... }` — it locks on the monitor of the specific object instance the method is called on.

### Synchronized Static Methods and the Class Object's Monitor

```java
class Counter {
    public static synchronized void resetGlobalCount() { /* ... */ }
}
```

A `synchronized` **static** method locks on the monitor of the **`Class` object** itself (e.g., `Counter.class`), not on any particular instance — meaning it coordinates across **all** instances of that class simultaneously, since there's only one `Class` object per class, shared JVM-wide.

### At the Bytecode Level

When Java source code is compiled, `synchronized` blocks are translated into bytecode instructions `monitorenter` and `monitorexit`, which the JVM uses to actually acquire and release the object's monitor at the lowest level. Synchronized **methods** don't use these explicit bytecode instructions directly — instead, the method itself is flagged with an `ACC_SYNCHRONIZED` access modifier in the compiled class file, and the JVM automatically performs the monitor acquire/release around the method's execution.

---

## Internal Working

Step-by-step of what happens when a thread executes `synchronized(lock) { ... }`:

1. The thread evaluates `lock`'s monitor entry count (every monitor internally tracks a count, supporting **reentrancy** — see the note below).
2. If the monitor's owner is `null` (unowned) or **already this same thread** (reentrant case), the thread becomes/remains the owner, and the entry count is incremented.
3. If the monitor is owned by a **different** thread, this thread is blocked (added to the monitor's "entry set," conceptually) until the current owner releases it.
4. The thread executes the code inside the `synchronized` block.
5. Upon exiting the block, the entry count is decremented; if it reaches zero, the monitor is fully released, and the JVM/OS may wake up one of the waiting threads to try acquiring it next.

### A Note on Reentrancy

Java monitors are **reentrant** — if a thread already holds a monitor, it can enter another `synchronized` block/method that uses the **same** monitor again without blocking on itself:

```java
public synchronized void outer() {
    inner(); // fine! Same thread already holds 'this' monitor
}

public synchronized void inner() {
    // ...
}
```

If monitors were **not** reentrant, this exact common pattern (one synchronized method calling another synchronized method on the same object) would cause the thread to deadlock against itself — reentrancy prevents this.

---

## Memory Diagram

```
        Object "lock" (every object has this structure)
   -----------------------------------------------
   |  Monitor:                                     |
   |    owner: null (or a specific Thread)          |
   |    entry count: 0 (increments per reentry)     |
   |    entry set: [threads waiting to acquire]     |
   |    wait set: [threads that called wait()]      |
   -----------------------------------------------

   Thread A: synchronized(lock) { ... }
       -> owner = Thread A, entry count = 1

   Thread B: synchronized(lock) { ... }  (while A holds it)
       -> Thread B added to entry set, BLOCKED

   Thread A exits block -> entry count = 0, owner = null
       -> Thread B removed from entry set, becomes owner
```

---

## Execution Flow

```
Thread A reaches synchronized(lock) { ... }
        |
        v
Is lock's monitor currently unowned, or owned by Thread A already?
        |
   -----------------------------
   |                           |
  YES                         NO (owned by Thread B)
   |                           |
   v                           v
Thread A acquires it,    Thread A added to entry set,
entry count incremented   enters BLOCKED state
   |                           |
   v                           v
Executes block            Waits until Thread B releases
   |                       the monitor
   v                           |
Exits block, entry count       |
decremented; if 0, monitor     |
released, owner = null         |
   |                           |
   +----------> Thread A (or another blocked thread)
                acquires the now-free monitor
```

---

## Code Example

```java
public class MonitorLockDemo {

    static class Counter {
        private int count = 0;

        public synchronized void increment() {
            count++;
            outerCheck(); // demonstrates reentrancy: calling another synchronized method on 'this'
        }

        public synchronized void outerCheck() {
            // Same thread already owns 'this' monitor from increment() -- no blocking here!
            System.out.println(Thread.currentThread().getName() + " reentered monitor. Count: " + count);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        Runnable task = () -> {
            for (int i = 0; i < 3; i++) {
                counter.increment();
            }
        };

        Thread t1 = new Thread(task, "Thread-A");
        Thread t2 = new Thread(task, "Thread-B");

        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("Final count: " + counter.count);
    }
}
```

### Line-by-line explanation

1. `increment()` is `synchronized` — it acquires `this` object's monitor before running.
2. Inside `increment()`, it calls `outerCheck()`, which is **also** `synchronized` on the same `this` object — because monitors are **reentrant**, the same thread can enter this second synchronized method without blocking on itself.
3. `t1` and `t2` both call `increment()` concurrently on the **same** `counter` instance — since both methods synchronize on the same object's monitor, their executions are properly serialized (one thread completes its full `increment()` + `outerCheck()` sequence before the other can start theirs on this same object).
4. Final count reliably prints `6` (3 increments × 2 threads), demonstrating correct mutual exclusion via the shared monitor.

---

## Dry Run

| Step | Thread | Action |
|---|---|---|
| 1 | Thread-A | Calls `increment()`, acquires `counter`'s monitor (entry count 1) |
| 2 | Thread-A | Calls `outerCheck()` — same thread, reentrant, entry count 2 |
| 3 | Thread-A | `outerCheck()` returns, entry count 1 |
| 4 | Thread-A | `increment()` returns, entry count 0, monitor released |
| 5 | Thread-B | (Was blocked if it tried during steps 1-4) Now acquires monitor, repeats the same sequence |
| ... | | Process repeats for all 3 iterations per thread |
| Final | Main | Prints "Final count: 6" |

---

## Common Mistakes

1. **Assuming different objects share a monitor just because they're the "same" logical data.**
   - Two separate `Counter` instances have two **completely separate** monitors — synchronizing on one instance provides zero protection for another instance, even if they conceptually represent "the same kind of thing."

2. **Synchronizing on a mutable reference that can change.**
   - E.g., `synchronized(someField)` where `someField` might be reassigned to point to a different object — this breaks the mutual exclusion guarantee, since different threads might end up synchronizing on different actual objects over time. Always synchronize on a stable, unchanging reference (often a `final` field created specifically as a lock).

3. **Forgetting that `synchronized` static methods lock the `Class` object, not an instance.**
   - Mixing a `synchronized` instance method and a `synchronized` static method, expecting them to coordinate with each other, is a mistake — they lock **different** monitors (the instance's monitor vs. the `Class` object's monitor) and provide **no** mutual exclusion between each other.

4. **Not realizing monitor reentrancy exists, and unnecessarily restructuring code to avoid "calling synchronized from synchronized."**
   - This is unnecessary — Java's monitors are explicitly designed to support this exact pattern safely.

---

## Production Usage

- **Every use of the `synchronized` keyword** in any production Java codebase relies directly on this monitor mechanism — it's not an optional, advanced feature; it's the foundational implementation detail behind one of Java's most commonly used concurrency tools.
- **Thread dump analysis** (see [[10 - Thread States]]) directly reports monitor ownership and waiting relationships — phrases like `"waiting to lock monitor 0x00007f..."` refer precisely to the mechanism described in this note, and are essential for diagnosing lock contention and deadlocks in production.
- **Framework internals** (connection pools, caches, thread-safe singletons) frequently rely on careful monitor-based synchronization, sometimes combined with `wait()`/`notify()` for more sophisticated coordination (see [[39 - Producer Consumer]]).

---

## Interview Questions

**Beginner:**
1. What is a monitor, in the context of Java concurrency?
2. Does every Java object have an associated monitor?
3. What does it mean for a thread to "own" a monitor?

**Intermediate:**
4. What monitor does a `synchronized` instance method lock on? What about a `synchronized` static method?
5. What does it mean for a monitor to be "reentrant," and why does this matter?
6. What JVM bytecode instructions correspond to entering and exiting a `synchronized` block?

**Advanced:**
7. Why would synchronizing on a mutable field reference (that could be reassigned) be a bug, rather than a safe pattern?
8. How does the monitor mechanism relate to `wait()` and `notify()`, beyond just mutual exclusion?

**Production-Level:**
9. How would you interpret a thread dump entry showing "waiting to lock monitor... held by Thread-X"?
10. Why is understanding monitor reentrancy important when reviewing or refactoring code with multiple `synchronized` methods calling each other?

---

## Follow-up Questions

- Can two threads hold the monitor of two **different** objects at the same time, with no blocking between them? *(Yes — monitors are entirely independent per object; there's no interaction between locking on `objectA` versus `objectB`.)*
- What happens if a thread throws an exception while holding a monitor inside a `synchronized` block? *(The monitor is still properly released as the exception propagates out of the block — Java guarantees this automatic release, similar to how a `finally` block would behave.)*
- Is there a limit to how many times a thread can reenter the same monitor? *(Practically, the entry count is just an integer counter with no meaningfully low limit for typical use — reentrancy depth is not a common practical concern.)*

---

## Best Practices

- Always synchronize on a stable, `final`, dedicated lock object (or `this`, when appropriate) — never on a reference that might be reassigned.
- Be intentional about whether you need instance-level (`synchronized` instance methods, or `synchronized(this)`) or class-level (`synchronized` static methods, or `synchronized(SomeClass.class)`) mutual exclusion — these are genuinely different monitors with different scopes.
- Take advantage of reentrancy freely when one synchronized method needs to call another on the same object — this is a safe, intended pattern, not something to avoid.
- When debugging lock contention via thread dumps, use your understanding of monitor ownership to trace exactly which object's monitor is causing a bottleneck.

---

## Performance Notes

- Acquiring an uncontended monitor (no other thread wants it) is very cheap in modern JVMs, thanks to optimizations like biased locking and lightweight locking (implementation details that have evolved across JVM versions).
- Contended monitors (multiple threads frequently competing for the same one) are where real performance costs and `BLOCKED`-state pileups occur — minimizing the scope and duration of synchronized sections directly reduces this contention (see [[27 - Critical Section]]).

---

## Comparison Table

| Aspect | Instance-level Monitor | Class-level Monitor |
|---|---|---|
| Locked via | `synchronized` instance method, or `synchronized(this)` | `synchronized` static method, or `synchronized(SomeClass.class)` |
| Scope | One specific object instance | Shared across ALL instances (and static context) of the class |
| Common use | Protecting an object's own mutable state | Protecting shared static/class-level state |

---

## Summary

- Every Java object has an associated **monitor** — a built-in lock supporting mutual exclusion and wait/notify coordination.
- `synchronized` blocks/methods acquire and release this monitor automatically, ensuring only one thread executes protected code for a given object at a time.
- Monitors are **reentrant** — a thread already holding a monitor can safely re-enter other synchronized code protected by the same monitor.
- Synchronized instance methods lock the object instance's monitor; synchronized static methods lock the `Class` object's monitor — these are distinct and don't provide mutual exclusion against each other.
- Understanding monitors is essential for correctly reading thread dumps and diagnosing lock contention in production.

---

## Cheat Sheet

| Concept | One-liner |
|---|---|
| Monitor | Built-in lock every object has |
| `synchronized(obj)` | Acquires `obj`'s monitor for the block's duration |
| Synchronized instance method | Locks `this`'s monitor |
| Synchronized static method | Locks the `Class` object's monitor |
| Reentrancy | Same thread can re-acquire a monitor it already holds |
| Bytecode | `monitorenter` / `monitorexit` (for blocks); `ACC_SYNCHRONIZED` flag (for methods) |

---

## Keywords

Monitor, Intrinsic Lock, Reentrancy, monitorenter, monitorexit, Mutual Exclusion, Class Object Lock

---

## Related Notes

[[22 - Synchronization]]
[[24 - Intrinsic Lock]]
[[25 - synchronized Keyword]]
[[40 - wait()]]
[[41 - notify()]]

---

## Practice Exercises

**Easy:**
1. Create a class with a `synchronized` method calling another `synchronized` method on the same object, and confirm no deadlock occurs (demonstrating reentrancy).
2. Create two separate instances of a class with a synchronized method, and confirm threads operating on different instances don't block each other.

**Medium:**
3. Create a class with both a `synchronized` instance method and a `synchronized` static method, and demonstrate (with timing/print statements) that they do NOT provide mutual exclusion against each other, since they lock different monitors.
4. Deliberately synchronize on a mutable field reference that gets reassigned mid-program, and observe how this breaks mutual exclusion.

**Hard:**
5. Write a program with 3 levels of reentrant synchronized method calls (method A calls B calls C, all synchronized on the same object), and add logging to trace the entry count conceptually via print statements at each level.

---

## Mini Project

**Project: Monitor Ownership Visualizer**

Build a small console application that:
1. Creates a shared object used as a lock.
2. Spawns several threads that each try to synchronize on this object, doing a small unit of work while holding it.
3. Logs, for each thread: when it requests the lock, when it acquires it, and when it releases it — including timestamps.
4. Produces a simple timeline output showing exactly how threads queue up for and take turns owning the monitor.

**Goal:** Turn the abstract "monitor ownership" concept into a directly observable, logged sequence of events, reinforcing exactly how mutual exclusion plays out over time.

---

## Common Interview Trap

**Trap Question:** *"If Class `Foo` has both a `synchronized` instance method `doWork()` and a `synchronized` static method `doGlobalWork()`, does calling `doWork()` on one thread block another thread from calling `doGlobalWork()` at the same time?"*

The trap is assuming "synchronized" always means "the same lock." In reality, these two methods lock **completely different monitors**: `doWork()` locks the specific object instance's monitor, while `doGlobalWork()` locks the `Class` object's monitor (`Foo.class`) — these are two entirely separate locks with no relationship to each other. So **no**, calling one does **not** block the other; a thread can be inside `doWork()` on one instance while another thread is simultaneously inside `doGlobalWork()`, with zero mutual exclusion between them. This is a frequently tested, subtle distinction.

---

## Frequently Asked Questions

**Q: Can I synchronize on any object, or only specially designated "lock" objects?**
A: Any object can serve as a lock — every object has a monitor by default. It's common practice to create a small, dedicated `private final Object lock = new Object();` specifically for this purpose, for clarity and to avoid accidental external interference.

**Q: Does synchronizing on a `String` literal work the same way as any other object?**
A: Technically yes, but it's discouraged — string literals are interned and potentially shared across unrelated parts of your program (or even other libraries), which can lead to surprising, unintended synchronization between unrelated code that happens to use the same literal value.

**Q: Is there a limit to how many threads can be in a monitor's "entry set" waiting to acquire it?**
A: No practical limit imposed by the JVM itself, though a very large number of waiting threads suggests a serious contention/design problem worth addressing.

**Q: Does exiting a `synchronized` block via a `return` statement still properly release the monitor?**
A: Yes — the JVM guarantees the monitor is released regardless of how the block is exited (normal completion, `return`, `break`, or an exception propagating out).

---

## Revision Checklist

- [ ] I understand that every Java object has an associated monitor.
- [ ] I understand what "owning" a monitor means and how mutual exclusion works.
- [ ] I understand the difference between instance-level and class-level (static) synchronized locking.
- [ ] I understand monitor reentrancy and why it prevents self-deadlock.
- [ ] I can correctly interpret "waiting to lock monitor" language in a thread dump.

---

## Flashcards

**Q:** What is a monitor in Java?
**A:** A built-in lock, associated with every object, providing mutual exclusion and wait/notify coordination.

**Q:** What monitor does a `synchronized` static method lock?
**A:** The `Class` object's monitor (shared across all instances).

**Q:** What monitor does a `synchronized` instance method lock?
**A:** The specific object instance's (`this`'s) monitor.

**Q:** What does "reentrant" mean in the context of monitors?
**A:** A thread that already owns a monitor can acquire it again (e.g., via a nested synchronized call) without blocking on itself.

**Q:** What JVM bytecode instructions implement `synchronized` blocks?
**A:** `monitorenter` and `monitorexit`.

---

## References

- Oracle Java Tutorials — Synchronization: https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html
- JLS (Java Language Specification) — Chapter 17: Threads and Locks (Monitor semantics)
- JVM Specification — `monitorenter`/`monitorexit` bytecode instructions
