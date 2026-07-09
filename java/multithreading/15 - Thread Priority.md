# Thread Priority

## Objective

By the end of this note, you will understand:

- The concrete Java API for setting and reading thread priority
- The valid priority range and what the constants mean
- How priority is inherited between threads
- Why, in practice, professional Java developers almost never touch this API

This note assumes you've read [[11 - Thread Scheduler]], which explains **why** priority is only a weak hint. This note focuses on the **concrete API mechanics** — the "how do I use it, and what exactly does it do" layer.

---

## Why should I learn this?

Even though thread priority is rarely used in real production code, it's still a commonly asked interview topic, because it tests whether you understand the difference between "what an API lets you configure" and "what that configuration actually guarantees." Knowing the mechanics precisely — the valid range, the constants, the inheritance behavior — rounds out your understanding from [[11 - Thread Scheduler]] with hands-on API knowledge.

---

## Beginner Explanation

Every Java thread has an integer "priority" value between 1 and 10. You can think of it as a **suggestion slip** you attach to a thread when handing it to the OS: "Hey, if you get a choice, maybe prefer running this one a bit more often." The OS is free to take this suggestion seriously, mostly ignore it, or interpret it in its own way — Java doesn't enforce anything beyond making the suggestion.

---

## Real World Analogy

**Airport Boarding Analogy**

Airlines often have "priority boarding" for certain ticket classes. A priority boarding pass is a **hint** to the gate staff: "let this passenger board a bit earlier if convenient." But it's not an absolute guarantee — gate staff can still call other groups, handle exceptions, or adjust based on real-time circumstances (a family needing help, a security matter, etc.). Priority boarding influences the _general tendency_ of who boards first, without providing a hard, unbreakable rule.

Java's thread priority works exactly the same way: a suggestion about general tendency, not an unbreakable rule.

---

## Technical Explanation

### The Priority Constants and Range

```java
public class Thread {
    public static final int MIN_PRIORITY = 1;
    public static final int NORM_PRIORITY = 5;
    public static final int MAX_PRIORITY = 10;
}
```

- Valid priority values are integers from **1 (lowest)** to **10 (highest)**.
- The **default priority for any new thread is 5** (`NORM_PRIORITY`), unless explicitly changed.

### Setting and Getting Priority

```java
Thread t = new Thread(() -> { /* work */ });

t.setPriority(8);              // set a custom priority (must be 1-10)
int p = t.getPriority();       // read the current priority
```

Attempting to set a priority outside the range 1-10 throws `IllegalArgumentException`:

```java
t.setPriority(15); // throws IllegalArgumentException
```

### Priority Inheritance

When a new thread is created, it **inherits the priority of the thread that created it**, unless explicitly overridden:

```java
Thread parent = new Thread(() -> {
    System.out.println("Parent priority: " + Thread.currentThread().getPriority());

    Thread child = new Thread(() -> {
        System.out.println("Child priority: " + Thread.currentThread().getPriority());
    });
    child.start(); // inherits parent's priority by default
});

parent.setPriority(9);
parent.start();
// Output: Parent priority: 9
//         Child priority: 9  (inherited, since child didn't call setPriority())
```

### How the JVM Maps Priority to the OS

Internally, when the JVM requests a native OS thread, it attempts to translate Java's 1-10 priority scale into whatever native priority system the underlying OS uses (which may have a completely different range or granularity — e.g., Windows has a small number of discrete priority classes, while some Unix-like systems use a wider numeric range with different semantics for "nice" values). This translation is **implementation-specific** and is a major reason priority behavior varies across platforms — reinforcing the core lesson from [[11 - Thread Scheduler]].

### Class Priority vs. Thread Group Priority

A thread's effective priority can also be capped by its **thread group's maximum priority** (`ThreadGroup.getMaxPriority()`) — if you try to set a thread's priority higher than its thread group allows, it will be silently capped rather than throwing an exception. Thread groups are a largely legacy feature today (see [[17 - Thread Groups]]), but this interaction is worth knowing for completeness and for the rare legacy codebase where it matters.

---

## Internal Working

Step-by-step of what happens when you set a thread's priority:

1. You call `thread.setPriority(n)`, where `n` is between 1 and 10.
2. The JVM validates `n` is in range; if not, it throws `IllegalArgumentException`.
3. The JVM also checks the thread's containing `ThreadGroup`'s max priority; if `n` exceeds that cap, the JVM silently reduces the effective value to the group's maximum.
4. The JVM stores this priority value and, when the thread is (or becomes) live, passes a translated version of it to the OS as a scheduling hint.
5. From that point forward, the OS scheduler may (or may not, depending on the OS and its current scheduling algorithm/load) factor this hint into its decisions about which `RUNNABLE` thread to run next.

---

## Memory Diagram

```
        Thread object
   -----------------------------
   |  priority: int (1-10)      |  <- stored on the Thread object itself
   -----------------------------
              |
              v
   JVM translates to native OS
   priority representation
   (varies by OS! not 1:1)
              |
              v
   OS scheduler MAY use this as
   one input among many when
   deciding which thread runs next
```

---

## Execution Flow

```
new Thread(task)
        |
        v
Inherits creating thread's priority (default: 5, if creator is main())
        |
        v
Optionally: thread.setPriority(n)   [1 <= n <= 10, else IllegalArgumentException]
        |
        v
Optionally capped by ThreadGroup's max priority
        |
        v
thread.start()
        |
        v
JVM translates priority to OS-native hint when requesting scheduling
        |
        v
OS scheduler factors this in loosely (varies by OS/JVM implementation)
```

---

## Code Example

```java
public class ThreadPriorityDemo {

    public static void main(String[] args) {

        System.out.println("Main thread priority: " + Thread.currentThread().getPriority()); // 5 by default

        Thread t1 = new Thread(() -> {
            System.out.println("t1 inherited priority: " + Thread.currentThread().getPriority());
        });
        t1.start(); // t1 inherits main's priority (5), since we never called setPriority on it

        Thread t2 = new Thread(() -> {
            System.out.println("t2 priority: " + Thread.currentThread().getPriority());
        });
        t2.setPriority(Thread.MAX_PRIORITY); // explicitly set to 10
        t2.start();

        try {
            Thread t3 = new Thread(() -> {});
            t3.setPriority(20); // invalid range
        } catch (IllegalArgumentException e) {
            System.out.println("Caught expected exception: " + e.getMessage());
        }
    }
}
```

### Line-by-line explanation

1. `Thread.currentThread().getPriority()` in `main()` — prints `5`, the default (`NORM_PRIORITY`) for the main thread.
2. `t1` is created without calling `setPriority()`, so it **inherits** priority `5` from `main` (its creating thread).
3. `t2.setPriority(Thread.MAX_PRIORITY)` — explicitly sets `t2`'s priority to `10`.
4. Attempting `t3.setPriority(20)` — throws `IllegalArgumentException`, since 20 is outside the valid 1-10 range.
5. Note that regardless of these priority values, as covered in [[11 - Thread Scheduler]], the **actual observed execution order** of `t1` and `t2` is not reliably determined by these settings alone — this example demonstrates the API mechanics, not a guarantee of behavior.

---

## Dry Run

|Step|Thread|Action|Priority|
|---|---|---|---|
|1|main|Prints own priority|5 (default)|
|2|main|Creates t1 (no explicit priority set)|t1 inherits 5|
|3|main|Creates t2, calls `setPriority(10)`|t2 = 10|
|4|main|Creates t3, calls `setPriority(20)`|Throws `IllegalArgumentException`|

---

## Common Mistakes

1. **Assuming a higher priority thread will always run before a lower priority one.**
    
    - As established in [[11 - Thread Scheduler]], this is only a hint — never a guarantee, especially on multi-core systems.
2. **Setting priority values outside 1-10.**
    
    - Always causes `IllegalArgumentException`; use the named constants (`MIN_PRIORITY`, `NORM_PRIORITY`, `MAX_PRIORITY`) or values strictly within that range.
3. **Forgetting that priority is inherited from the creating thread, not fixed at some universal default.**
    
    - If a thread with a customized (non-default) priority creates further threads, those children inherit **that** priority, not automatically `NORM_PRIORITY`, unless explicitly reset.
4. **Using priority as a substitute for real synchronization.**
    
    - E.g., trying to "ensure" one thread finishes before another purely via priority differences. This is unreliable — use `join()`, latches, or proper synchronization instead (see [[20 - Join()]], [[54 - CountDownLatch]]).

---

## Production Usage

- **Rarely used directly** in typical business application code — most production Java systems leave every thread at the default `NORM_PRIORITY` and rely on proper thread pool sizing (see [[44 - Thread Pool]], [[72 - Performance Tuning]]) rather than priority tuning for performance.
- **Some specialized systems** (e.g., certain real-time or latency-sensitive components) may experiment with priority hints as one of several tuning levers, but typically combine this with OS-level tools for any serious guarantees, since Java's own priority API alone is considered too weak/unreliable for hard real-time requirements.
- **Framework internals**: occasionally, a framework might slightly deprioritize genuinely low-importance background housekeeping threads, but this is uncommon and always treated as a soft optimization, never a correctness mechanism.

---

## Interview Questions

**Beginner:**

1. What is the valid range for Java thread priority values?
2. What is the default priority for a new thread?
3. What exception is thrown if you try to set an invalid priority value?

**Intermediate:** 4. Does a child thread inherit its priority from its parent thread, or does it always default to `NORM_PRIORITY`? 5. What are the three named priority constants in the `Thread` class, and their integer values? 6. Can a `ThreadGroup` limit the maximum priority a thread within it can have?

**Advanced:** 7. Why does Java's priority value not map consistently to OS-level priority values across different platforms? 8. Why is thread priority generally considered an unreliable mechanism for controlling execution order in production systems?

**Production-Level:** 9. In what rare, specialized scenario might a team consider using thread priority as part of a broader performance tuning strategy? 10. Why do most production Java systems leave thread priority at its default value entirely?

---

## Follow-up Questions

- If I create a thread inside another thread that has priority 3, and never call `setPriority()` on the child, what priority will the child have? _(3, inherited from its parent.)_
- Is there a way to reliably guarantee one thread always runs before another using only priority? _(No — use explicit synchronization mechanisms like `join()` or `CountDownLatch` instead.)_
- Does changing a thread's priority after it has already started have any effect? _(Yes, it can be changed at any time during a thread's life via `setPriority()`, and the new value takes effect going forward — but as always, its actual influence on scheduling remains just a hint.)_

---

## Best Practices

- Leave thread priority at the default (`NORM_PRIORITY`) unless you have a very specific, well-understood reason to change it.
- Never rely on priority for correctness or ordering guarantees — use proper synchronization tools instead.
- If you do experiment with priority, treat it purely as a minor performance tuning lever, and always validate its actual effect empirically on your target deployment environment (see [[11 - Thread Scheduler]]'s discussion of environment-dependent behavior).

---

## Performance Notes

- Adjusting thread priority is unlikely to produce meaningful, reliable performance improvements for typical business applications — proper thread pool sizing (see [[72 - Performance Tuning]]) is a far higher-leverage lever.
- Priority tuning is more relevant (though still limited) in specialized low-level or real-time contexts, often combined with native/OS-level tools beyond what the standard Java API provides.

---

## Comparison Table

|Constant|Value|Typical Use|
|---|---|---|
|`Thread.MIN_PRIORITY`|1|Rarely used explicitly; lowest hint|
|`Thread.NORM_PRIORITY`|5|Default for all threads unless changed|
|`Thread.MAX_PRIORITY`|10|Rarely used explicitly; highest hint|

---

## Summary

- Java thread priority is an integer from 1 (`MIN_PRIORITY`) to 10 (`MAX_PRIORITY`), with 5 (`NORM_PRIORITY`) as the default.
- New threads inherit their priority from the thread that created them, unless explicitly overridden.
- Setting an invalid priority value (outside 1-10) throws `IllegalArgumentException`.
- A thread's effective priority can be capped by its `ThreadGroup`'s maximum priority.
- In practice, priority is only a weak scheduling hint (see [[11 - Thread Scheduler]]) and is rarely used in professional production code — proper synchronization and thread pool sizing are the correct tools for correctness and performance, respectively.

---

## Cheat Sheet

|API|Behavior|
|---|---|
|`Thread.MIN_PRIORITY`|1|
|`Thread.NORM_PRIORITY`|5 (default)|
|`Thread.MAX_PRIORITY`|10|
|`setPriority(int)`|Sets priority; throws `IllegalArgumentException` if outside 1-10|
|`getPriority()`|Returns current priority|
|Inheritance|New threads inherit their creator's priority by default|

---

## Keywords

Thread Priority, MIN_PRIORITY, NORM_PRIORITY, MAX_PRIORITY, setPriority, getPriority, IllegalArgumentException, ThreadGroup, Priority Inheritance

---

## Related Notes

[[11 - Thread Scheduler]] [[17 - Thread Groups]] [[04 - Thread Creation]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Print the default priority of the main thread and of a newly created thread.
2. Set a thread's priority to `Thread.MAX_PRIORITY` and confirm via `getPriority()`.

**Medium:** 3. Create a thread with a custom priority, have it create a child thread without setting priority explicitly, and confirm the child inherited the parent's custom priority. 4. Attempt to set an invalid priority value and catch the resulting exception, printing a friendly message.

**Hard:** 5. Combine this note's API knowledge with [[11 - Thread Scheduler]]'s benchmark style: create high- and low-priority threads doing equal CPU-bound work, and empirically measure whether priority has any observable effect on your specific machine/OS.

---

## Mini Project

**Project: Priority Inheritance Tree Visualizer**

Build a small console application that:

1. Creates a "grandparent" thread with a custom priority (e.g., 8).
2. Has it create a "parent" thread (without setting priority explicitly).
3. Has the parent create a "child" thread (also without setting priority explicitly).
4. Prints each thread's actual priority, visually confirming the inheritance chain: grandparent's custom priority flows down to parent and child.

**Goal:** Build direct, hands-on confirmation of Java's priority inheritance rule, reinforcing the mechanics covered in this note.

---

## Common Interview Trap

**Trap Question:** _"If I never call `setPriority()` on any thread in my application, what priority will they all have?"_

The trap is assuming every thread defaults to `NORM_PRIORITY` (5) directly. The more precise answer: **every thread inherits its priority from its creating thread**, and since the very first thread (`main`) starts at `NORM_PRIORITY` by JVM default, in a typical application where no one ever calls `setPriority()`, the _practical_ effect is indeed that all threads end up at priority 5 — but the _mechanism_ is inheritance, not some universal, independent default applied to each thread individually. This distinction matters if any thread in the chain **does** call `setPriority()` — every subsequently created thread from that point in the chain will inherit the modified value, not silently revert to 5.

---

## Frequently Asked Questions

**Q: Does setting a very high priority guarantee low latency for a thread?** A: No — it's only a hint, and low-latency guarantees typically require dedicated real-time OS features or native tools well beyond Java's standard priority API.

**Q: Can I query a thread's priority from another thread?** A: Yes — `someOtherThread.getPriority()` can be called from any thread that has a reference to the target `Thread` object.

**Q: Is priority preserved if a thread is paused (e.g., sleeping) and resumes?** A: Yes — priority is a persistent property of the `Thread` object itself, unaffected by state transitions like sleeping or waiting.

**Q: Does the `ExecutorService` framework let you configure thread priority for pooled worker threads?** A: Yes, indirectly — by supplying a custom `ThreadFactory` when creating the executor, which can set priority (and daemon status) on each worker thread it creates (see [[44 - Thread Pool]], [[45 - Executor Framework]]).

---

## Revision Checklist

- [ ] I know the valid range (1-10) and default value (5) for thread priority.
- [ ] I understand that new threads inherit priority from their creator by default.
- [ ] I know what exception is thrown for an invalid priority value.
- [ ] I understand that a `ThreadGroup` can cap a thread's effective maximum priority.
- [ ] I understand why priority should never be relied upon for correctness.

---

## Flashcards

**Q:** What is the valid range for Java thread priority? **A:** 1 (`MIN_PRIORITY`) to 10 (`MAX_PRIORITY`).

**Q:** What is the default thread priority? **A:** 5 (`NORM_PRIORITY`).

**Q:** How does a new thread determine its initial priority? **A:** It inherits the priority of the thread that created it.

**Q:** What exception is thrown for an out-of-range priority value? **A:** `IllegalArgumentException`.

**Q:** Can a `ThreadGroup` limit a thread's maximum priority? **A:** Yes — via `ThreadGroup.getMaxPriority()`, silently capping any higher requested value.

---

## References

- Oracle Java Tutorials — Thread Priority: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- `java.lang.Thread` API Documentation (`setPriority`, `getPriority`, priority constants)
- `java.lang.ThreadGroup` API Documentation