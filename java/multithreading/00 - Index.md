# Java Multithreading — Master Handbook

## Objective

This is the entry point of your complete Java Multithreading learning vault. By the end of this handbook, you will understand multithreading from the absolute basics (what a thread even is) all the way to production-grade concerns like debugging thread dumps, tuning thread pools, and understanding Virtual Threads (Project Loom).

This vault is built to be read **both linearly** (as a course) and **non-linearly** (as a revision tool before interviews), thanks to Obsidian's linking system.

---

## Why should I learn this?

Almost every real Java backend system — web servers, databases, message brokers, batch processors, trading engines — depends on concurrency. A Java developer who does not understand threading will:

- Struggle to explain why an application is slow under load
- Misuse `synchronized`, causing deadlocks or poor throughput
- Fail mid-to-senior level interviews (concurrency is one of the most tested areas)
- Be unable to debug production incidents involving thread starvation, contention, or memory visibility bugs

Multithreading exists because modern CPUs have multiple cores, and a single-threaded program can only ever use one of them at a time. Learning this topic well is what separates a junior Java developer from a senior engineer who can be trusted with production systems.

---

## How to use this vault

1. Start from **Section 1: Foundations** and move sequentially the first time through.
2. Use `[[Page Name]]` links inside every note to jump to related concepts as questions come up.
3. Before an interview, jump directly to **Section 14: Interview & Revision** and use the Cheat Sheets + Flashcards in each note.
4. Every note follows the same structure (Objective → Beginner Explanation → Analogy → Technical Deep Dive → Code → Interview Questions → Cheat Sheet), so you always know where to find what you need.

---

## Vault Map

### Section 1 — Foundations
- [[01 - What is Multithreading]]
- [[02 - Process vs Thread]]
- [[03 - Why Multithreading Exists]]

### Section 2 — Creating & Running Threads
- [[04 - Thread Creation]]
- [[05 - Runnable]]
- [[06 - Callable]]
- [[07 - Future]]
- [[08 - FutureTask]]

### Section 3 — Thread Lifecycle & Control
- [[09 - Thread Lifecycle]]
- [[10 - Thread States]]
- [[11 - Thread Scheduler]]
- [[12 - Context Switching]]
- [[13 - Daemon Thread]]
- [[14 - User Thread]]
- [[15 - Thread Priority]]
- [[16 - Thread Naming]]
- [[17 - Thread Groups]]
- [[18 - Sleep()]]
- [[19 - Yield()]]
- [[20 - Join()]]
- [[21 - Interrupt()]]

### Section 4 — Synchronization Fundamentals
- [[22 - Synchronization]]
- [[23 - Monitor Lock]]
- [[24 - Intrinsic Lock]]
- [[25 - synchronized Keyword]]
- [[26 - Race Condition]]
- [[27 - Critical Section]]
- [[28 - Visibility Problem]]
- [[29 - Volatile]]
- [[30 - Atomic Variables]]
- [[31 - CAS]]

### Section 5 — Advanced Locks
- [[32 - ReentrantLock]]
- [[33 - ReadWriteLock]]
- [[34 - StampedLock]]
- [[35 - Condition]]

### Section 6 — Concurrency Problems
- [[36 - Deadlock]]
- [[37 - Livelock]]
- [[38 - Starvation]]
- [[39 - Producer Consumer]]

### Section 7 — Inter-Thread Communication
- [[40 - wait()]]
- [[41 - notify()]]
- [[42 - notifyAll()]]
- [[43 - BlockingQueue]]

### Section 8 — Thread Pools & Executors
- [[44 - Thread Pool]]
- [[45 - Executor Framework]]
- [[46 - ExecutorService]]
- [[47 - ScheduledExecutorService]]
- [[48 - Fork Join Pool]]
- [[49 - CompletableFuture]]
- [[50 - Parallel Streams]]

### Section 9 — Concurrent Collections & Coordination Utilities
- [[51 - Concurrent Collections]]
- [[52 - ConcurrentHashMap]]
- [[53 - CopyOnWriteArrayList]]
- [[54 - CountDownLatch]]
- [[55 - CyclicBarrier]]
- [[56 - Phaser]]
- [[57 - Semaphore]]
- [[58 - Exchanger]]
- [[59 - LockSupport]]

### Section 10 — Thread-Local Storage
- [[60 - ThreadLocal]]
- [[61 - InheritableThreadLocal]]

### Section 11 — Java Memory Model & Hardware
- [[62 - Java Memory Model]]
- [[63 - Happens Before]]
- [[64 - Memory Consistency]]
- [[65 - False Sharing]]
- [[66 - CPU Cache]]
- [[67 - Garbage Collection and Threads]]

### Section 12 — JVM Internals & Debugging
- [[68 - JVM Thread Management]]
- [[69 - Thread Dump Analysis]]
- [[70 - Debugging Multithreading]]
- [[71 - Profiling Threads]]
- [[72 - Performance Tuning]]
- [[73 - Common Production Bugs]]

### Section 13 — Framework & Real-World Usage
- [[74 - Spring Boot Threading Model]]
- [[75 - Tomcat Thread Pool]]
- [[76 - Kafka Consumer Threads]]
- [[77 - Async in Spring]]
- [[78 - Virtual Threads (Project Loom)]]
- [[79 - Structured Concurrency]]
- [[80 - Reactive vs Multithreading]]

### Section 14 — Interview & Revision
- [[81 - Interview Questions]]
- [[82 - Production Case Studies]]
- [[83 - Revision Notes]]
- [[84 - Glossary]]
- [[85 - Cheat Sheet]]

### Section 15 — Practice
- [[86 - 100 Coding Exercises]]
- [[87 - 50 Debugging Exercises]]
- [[88 - 25 Mini Projects]]
- [[89 - 20 Production Design Problems]]
- [[90 - Final Interview Revision Guide]]

---

## Suggested Learning Path (if you are a beginner)

```
Section 1 (Foundations)
      |
      v
Section 2 (Creating Threads)
      |
      v
Section 3 (Lifecycle & Control)
      |
      v
Section 4 (Synchronization Basics)
      |
      v
Section 6 (Concurrency Problems) <-- understand WHY locks matter
      |
      v
Section 5 (Advanced Locks)
      |
      v
Section 7 (wait/notify, BlockingQueue)
      |
      v
Section 8 (Executors, Thread Pools) <-- most used in real jobs
      |
      v
Section 9 (Concurrent Collections & Utilities)
      |
      v
Section 11 (Java Memory Model) <-- the "why" behind everything above
      |
      v
Section 12 (JVM Internals & Debugging)
      |
      v
Section 13 (Frameworks: Spring, Tomcat, Kafka, Loom)
      |
      v
Section 14 & 15 (Interview Prep + Practice)
```

---

## Status Tracker

Use this table to track your progress. Update the "Status" column as you complete each note.

| # | Note | Status |
|---|------|--------|
| 00 | Index | ✅ Done |
| 01 | What is Multithreading | ⬜ Pending |
| 02 | Process vs Thread | ⬜ Pending |
| 03 | Why Multithreading Exists | ⬜ Pending |
| ... | ... | ⬜ Pending |
| 90 | Final Interview Revision Guide | ⬜ Pending |

*(Full 90-row table will be maintained as notes are generated — this is a living document.)*

---

## Keywords

Multithreading, Concurrency, JVM, Thread, Process, Synchronization, Java Memory Model, Executor Framework, Virtual Threads, Production Debugging

---

## Related Notes

[[01 - What is Multithreading]]
[[68 - JVM Thread Management]]
[[81 - Interview Questions]]

---

## References

- Oracle Java Documentation — Concurrency: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- java.util.concurrent Package Summary (Oracle Docs)
- Relevant JEPs: JEP 444 (Virtual Threads), JEP 428 (Structured Concurrency)
