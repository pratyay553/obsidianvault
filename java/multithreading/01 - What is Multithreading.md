# What is Multithreading?

## Objective

By the end of this note, you will understand:

- What a "thread" actually is, in plain English
- What "multithreading" means in Java specifically
- How multithreading relates to your computer's CPU
- Why this concept is the foundation for everything else in this vault

This note assumes **zero prior knowledge** of concurrency. If you have never written multithreaded code before, this is the correct starting point.

---

## Why should I learn this?

Every Java application you have ever run — even a simple `System.out.println("Hello World")` program — already runs on a thread. You just never had to think about it, because it was the **only** thread running.

The moment your application needs to do more than one thing at the same time — like a web server handling 500 users simultaneously, or a mobile app downloading a file while keeping its UI responsive — you need multithreading.

Java provides built-in support for multithreading because:

1. **Modern CPUs have multiple cores.** A single thread can only ever use one core. If your program only creates one thread, you are wasting the other cores sitting idle in the CPU.
2. **Real-world programs need to multitask.** A server handling one request at a time would be unusably slow.
3. **Responsiveness matters.** Applications that block on slow operations (file I/O, network calls, database queries) need other threads to keep working while they wait.

Without multithreading, Java applications would not be able to scale to handle real-world, production-level traffic.

---

## Beginner Explanation

Imagine you are the only person working in a small shop. You can only serve **one customer at a time**. If a customer takes 10 minutes to decide what they want, every other customer waiting in line is stuck doing nothing but waiting.

Now imagine you hire 3 more workers. Now 4 customers can be served **at the same time**, by 4 different workers. This is exactly what multithreading does inside a computer program.

- **A thread** is like one "worker" inside your program.
- **Multithreading** means your program has multiple workers doing tasks, instead of just one.
- Each worker (thread) can do its own job independently, but they might sometimes need to share the same resources (like a shared cash register), which is where things get tricky — but we'll get to that in later notes.

In Java, when you run any program, the JVM (Java Virtual Machine) automatically creates **at least one thread**, called the **main thread**. This is the thread that runs your `main()` method. Multithreading is when you create **additional threads** on top of this main thread, so multiple parts of your program can execute independently and, on multi-core CPUs, truly _simultaneously_.

---

## Real World Analogy

**Restaurant Analogy**

Think of a restaurant kitchen:

- **Single-threaded kitchen:** One chef does everything — takes the order, cooks the appetizer, cooks the main course, prepares the dessert, and serves it. Every other order has to wait until this one is 100% finished.
- **Multithreaded kitchen:** Multiple chefs work in parallel. One chef makes appetizers, another handles grilling, another does desserts. Orders get completed much faster because work happens **concurrently**.

Just like a restaurant kitchen, a Java application benefits massively from splitting work across multiple "chefs" (threads) instead of forcing one chef (single thread) to do everything sequentially.

---

## Technical Explanation

### What is a Thread, technically?

A **thread** is the smallest unit of execution within a process. Each thread has:

- Its own **program counter** (keeps track of which instruction to execute next)
- Its own **call stack** (stores method calls and local variables)
- Its own **thread-local storage** (optional, we'll cover this in [[60 - ThreadLocal]])
- **Shared access** to the process's heap memory (objects, static variables)

### How the JVM handles threads

When you start a Java program:

1. The **JVM starts**, and the operating system (OS) allocates a **process** for it.
2. The JVM creates the **main thread** automatically. This thread executes your `main()` method.
3. If your code creates new `Thread` objects and starts them, the JVM asks the **underlying OS** to create actual **native OS threads** (in the standard JVM thread model, Java threads map directly to OS threads — this is called **1:1 threading model**).
4. The **OS scheduler** — not the JVM — decides which thread gets CPU time and when. This is called **thread scheduling**, covered in detail in [[11 - Thread Scheduler]].

### CPU Involvement

A CPU core can only truly execute **one instruction stream at a time**. If you have a quad-core CPU:

- Up to 4 threads can run **truly in parallel** (one per core) at any given instant.
- If you have more threads than cores (which is extremely common — a server might have 200 threads but only 8 cores), the OS rapidly switches between threads, giving each a small time slice. This creates the _illusion_ of many things happening at once, even though, strictly, only a few are truly simultaneous. This switching is called **context switching**, covered in [[12 - Context Switching]].

### Concurrency vs Parallelism (Important distinction)

- **Concurrency**: Multiple tasks make progress over the same time period (may or may not run at the exact same instant). Think of it as _dealing with_ many things at once.
- **Parallelism**: Multiple tasks run at the **exact same instant**, on separate CPU cores. Think of it as _doing_ many things at once.

Multithreading enables both — depending on how many CPU cores are available.

---

## Internal Working

Here is what happens, step by step, when a Java program with multiple threads runs:

1. JVM starts → OS creates a process for the JVM.
2. JVM creates the **main thread** and begins executing `main()`.
3. Your code calls `new Thread(...)` — this creates a `Thread` object in the JVM's heap memory, but **does not** start it yet.
4. You call `.start()` on the thread object.
5. The JVM makes a system call to the OS, requesting a **new native thread**.
6. The OS allocates:
    - A separate **call stack** for this new thread
    - CPU scheduling metadata
7. The new thread is placed into the **Runnable** state (see [[10 - Thread States]]).
8. The **OS thread scheduler** decides when this thread actually gets CPU time, based on priority, fairness policies, and core availability.
9. Once scheduled, the thread executes its `run()` method independently of the main thread.
10. Multiple threads may now be executing concurrently — sharing the heap, but each with their own stack.

---

## Memory Diagram

```
                    JVM Process
        ---------------------------------
        |                               |
        |          Heap Memory          |
        |   (Shared by ALL threads)     |
        |   - Objects                   |
        |   - Static variables          |
        ---------------------------------
              |            |          |
              v            v          v
        ------------  ------------  ------------
        Main Thread   Worker-1      Worker-2
        ------------  ------------  ------------
        | Own Stack |  | Own Stack |  | Own Stack |
        | Own PC    |  | Own PC    |  | Own PC    |
        ------------  ------------  ------------
```

- **Heap** = shared memory. All threads can read/write shared objects here (this is where race conditions come from — see [[26 - Race Condition]]).
- **Stack** = private memory per thread. Local variables and method call frames live here — no other thread can access another thread's stack.

---

## Execution Flow

A simple mental model of execution flow for a 2-thread program:

```
main() starts
   |
   |-- creates Thread T1
   |-- calls T1.start()
   |
   |---------------------------|
   |  Main Thread continues     |   T1 begins running independently
   |  doing its own work        |   doing its own work
   |---------------------------|
   |
   v                            v
Main thread finishes      T1 finishes (maybe earlier,
its work                  maybe later — not guaranteed!)
```

**Key point:** There is **no guarantee** which thread finishes first. The OS scheduler decides execution order, and it can vary **every time you run the program**. This unpredictability is a core reason concurrency bugs are hard to catch — a bug might not show up for thousands of runs, then suddenly appear in production.

---

## Code Example

```java
public class MultithreadingDemo {

    public static void main(String[] args) {
        System.out.println("Main thread started: " + Thread.currentThread().getName());

        // Create a new thread
        Thread worker = new Thread(() -> {
            System.out.println("Worker thread running: " + Thread.currentThread().getName());
        });

        // Start the new thread
        worker.start();

        System.out.println("Main thread finished starting worker.");
    }
}
```

### Line-by-line explanation

1. `Thread.currentThread().getName()` — asks the JVM: "which thread is currently executing this line of code?" For the very first line, this will always print `main`.
2. `new Thread(() -> {...})` — creates a **new Thread object** in memory. This does **not** start execution yet. The lambda passed in is the code that will run inside this new thread (it implements the `Runnable` interface — covered in [[05 - Runnable]]).
3. `worker.start()` — this is the crucial line. Calling `.start()` tells the JVM to request a **real OS thread** and begin executing the `run()` logic **concurrently** with the main thread.
    - **Common beginner mistake**: calling `worker.run()` instead of `worker.start()`. This does NOT create a new thread — it just calls the method normally on the current thread. We'll cover this in Common Mistakes below.
4. The last `System.out.println` in `main()` may print **before or after** the worker thread's print statement — there's no guarantee of order.

---

## Dry Run

Let's trace through possible outputs of the code above.

**Possible Output 1:**

```
Main thread started: main
Main thread finished starting worker.
Worker thread running: Thread-0
```

**Possible Output 2:**

```
Main thread started: main
Worker thread running: Thread-0
Main thread finished starting worker.
```

Both outputs are **completely valid**. Why?

- After `worker.start()` is called, the JVM hands off scheduling to the OS.
- The OS might immediately schedule the worker thread (Output 2), or it might let the main thread continue running first (Output 1).
- This is determined by the OS scheduler, CPU load, and thread priorities at that exact moment — it is **non-deterministic**.

This unpredictability is the single most important concept to internalize as a beginner: **multithreaded code does not execute in a fixed, predictable order.**

---

## Common Mistakes

1. **Calling `.run()` instead of `.start()`**
    
    - `run()` just executes the code like a normal method call on the _same_ thread — no new thread is created.
    - `start()` actually creates a new thread and schedules it with the OS.
2. **Assuming order of execution**
    
    - Beginners often assume the main thread will always finish first, or the worker thread will always run immediately. Neither is guaranteed.
3. **Not understanding shared memory (leads to future bugs)**
    
    - Beginners often update shared variables from multiple threads without synchronization, leading to race conditions (see [[26 - Race Condition]]). This note doesn't cover synchronization yet, but keep this in mind as you continue.
4. **Creating too many threads**
    
    - Creating thousands of raw `Thread` objects for small tasks is expensive (each OS thread consumes memory, typically ~1MB stack by default) and can crash the application. This is why we use **Thread Pools** (see [[44 - Thread Pool]]) in real projects instead of manually created threads.

---

## Production Usage

In real production systems, multithreading is everywhere, even if you don't manually create `Thread` objects yourself:

- **Web servers (Tomcat, Jetty)**: Every incoming HTTP request is handled on its own thread from a thread pool, so the server can handle thousands of requests concurrently. See [[75 - Tomcat Thread Pool]].
- **Spring Boot applications**: Use thread pools internally for `@Async` methods, scheduled tasks, and request handling. See [[74 - Spring Boot Threading Model]].
- **Kafka Consumers**: Applications often spin up multiple consumer threads to process messages from different partitions in parallel. See [[76 - Kafka Consumer Threads]].
- **Batch processing systems**: Large datasets are split into chunks, each processed on a separate thread, to finish faster.
- **Payment/Banking systems**: Multiple transactions are processed concurrently, but with strict synchronization to avoid double-charging or balance corruption.

You will rarely write raw `new Thread()` code in a real production Java codebase — but understanding what's happening underneath is essential to using higher-level tools (`ExecutorService`, `CompletableFuture`, Spring's `@Async`) correctly and debugging issues when they go wrong.

---

## Interview Questions

**Beginner:**

1. What is a thread in Java?
2. What is the difference between a program, a process, and a thread?
3. What method do you call to start a new thread — `run()` or `start()`?
4. Does Java guarantee the order in which threads execute?

**Intermediate:** 5. What is the difference between concurrency and parallelism? 6. What happens internally when you call `Thread.start()`? 7. What part of memory is shared between threads, and what part is private to each thread?

**Advanced:** 8. How does the JVM map Java threads to OS-level threads? 9. What is the cost (memory/performance) of creating a large number of threads? 10. Why can't a single CPU core truly run two threads at the exact same nanosecond?

**Production-Level:** 11. How does a web server like Tomcat use multithreading to handle thousands of concurrent requests with a limited thread pool? 12. What issues can occur if too many threads are created in a production system, and how would you detect this in a running application (thread dump analysis)?

---

## Follow-up Questions

- If I call `start()` twice on the same thread object, what happens? _(Answer: `IllegalThreadStateException` is thrown — a thread cannot be started more than once.)_
- What is the default stack size for a new thread in the JVM, and can it be configured? _(Answer: typically 512KB–1MB depending on OS/JVM; configurable via `-Xss` flag or `Thread` constructor overload.)_
- If your CPU has 4 cores and you create 100 threads, can all 100 truly run "at once"? _(Answer: No — only up to 4 can run simultaneously; the rest wait via context switching.)_

---

## Best Practices

- Prefer high-level concurrency utilities (`ExecutorService`, `CompletableFuture`) over manually creating raw `Thread` objects in production code.
- Always give threads meaningful names (`Thread-0` is not useful in a thread dump during an incident) — see [[16 - Thread Naming]].
- Never assume execution order between threads unless you explicitly synchronize or coordinate it.
- Keep thread creation cost in mind — don't create a new thread for every tiny task; use pooling.

---

## Performance Notes

- **Memory**: Each thread consumes stack memory (default ~512KB–1MB). 1,000 raw threads could consume ~1GB of memory just in stacks.
- **CPU**: More threads than CPU cores does not mean more speed — it can actually _reduce_ performance due to excessive context switching overhead.
- **Context Switching Cost**: Switching between threads has a real CPU cost (saving/restoring registers, cache invalidation). This is why "more threads = faster" is a common **myth** — see [[72 - Performance Tuning]].

---

## Comparison Table

|Aspect|Single-Threaded Program|Multithreaded Program|
|---|---|---|
|Number of workers|1|2 or more|
|CPU core usage|Only 1 core used|Can use multiple cores|
|Execution order|Fully predictable|Non-deterministic|
|Complexity|Simple|Higher (needs synchronization)|
|Best for|Simple sequential tasks|Servers, real-time apps, parallel computation|
|Risk of bugs like race conditions|None|Possible if shared data isn't protected|

---

## Summary

- A **thread** is the smallest unit of execution in a program.
- **Multithreading** means running multiple threads concurrently within a single process.
- The JVM automatically creates a **main thread**; additional threads are created explicitly by your code.
- Threads share **heap memory** but have their own **private stack**.
- Thread execution order is **not guaranteed** — this is the source of most concurrency bugs.
- Real production systems rely heavily on multithreading (web servers, Kafka, Spring Boot) but almost always through higher-level abstractions like thread pools, not raw `Thread` objects.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|Thread|Smallest unit of execution inside a process|
|Main Thread|Automatically created by JVM to run `main()`|
|`start()`|Creates a new OS thread and begins execution|
|`run()`|Just a normal method call — no new thread created|
|Heap|Shared memory across all threads|
|Stack|Private memory per thread|
|Concurrency|Multiple tasks making progress over time|
|Parallelism|Multiple tasks executing at the exact same instant|

---

## Keywords

Thread, Process, JVM, Main Thread, Concurrency, Parallelism, OS Scheduler, Context Switching, Heap, Stack, `start()`, `run()`

---

## Related Notes

[[02 - Process vs Thread]] [[03 - Why Multithreading Exists]] [[04 - Thread Creation]] [[09 - Thread Lifecycle]] [[26 - Race Condition]]

---

## Practice Exercises

**Easy:**

1. Write a program that creates one thread and prints "Hello from thread" inside it.
2. Print the name of the main thread using `Thread.currentThread().getName()`.

**Medium:** 3. Create two threads, each printing numbers 1–5. Run the program 3 times and observe whether the output order changes. 4. Modify the code above to give each thread a custom name using the `Thread` constructor.

**Hard:** 5. Create 3 threads where each thread increments a _shared_ `int` counter 100,000 times (without synchronization). Run it and observe that the final value is often **not** 300,000. (This is your first hands-on encounter with a race condition — don't worry about fixing it yet; that's covered in [[26 - Race Condition]] and [[22 - Synchronization]].)

---

## Mini Project

**Project: Multi-Worker Greeting Printer**

Build a small program that:

1. Creates 5 threads, each representing a "worker."
2. Each worker prints its name and a greeting message.
3. The main thread prints a "All workers launched" message.
4. Run the program 5 times and write down the different orders of output you observe.

**Goal:** Internalize that thread execution order is non-deterministic — this is the most important beginner lesson in this entire topic.

---

## Common Interview Trap

**Trap Question:** _"If I call `t.run()` instead of `t.start()`, will the code still execute?"_

Most beginners think `run()` won't work at all — that's wrong. `run()` **will execute the code**, but it does so **on the current thread** (e.g., the main thread), not on a new thread. So the code technically works, but you get **zero concurrency benefit** — it just behaves like a regular method call. This is a very common interview trick question used to test real understanding versus memorized syntax.

---

## Frequently Asked Questions

**Q: Is the main thread a "special" thread in Java?** A: Not architecturally special — it's a normal thread — but it's automatically created and started by the JVM to execute `main()`, and the JVM typically waits for all non-daemon threads (including main) to finish before exiting.

**Q: Can I create a thread without extending `Thread` or implementing `Runnable`?** A: No — in classic Java threading, you must use one of these two approaches (or the newer virtual thread APIs covered in [[78 - Virtual Threads (Project Loom)]]).

**Q: Does more threads always mean better performance?** A: No. Beyond a certain point (related to CPU core count and task type), adding more threads can _hurt_ performance due to context-switching overhead and resource contention.

**Q: What happens if the main thread finishes before the worker thread?** A: The JVM process stays alive as long as at least one **non-daemon** thread is still running. So the JVM won't necessarily exit just because `main()` returns. See [[13 - Daemon Thread]].

---

## Revision Checklist

- [ ] I can explain what a thread is, in my own words, without technical jargon.
- [ ] I understand the difference between `start()` and `run()`.
- [ ] I understand that thread execution order is not guaranteed.
- [ ] I know that threads share heap memory but have private stacks.
- [ ] I can explain concurrency vs. parallelism with an example.
- [ ] I understand why creating too many raw threads is bad practice.

---

## Flashcards

**Q:** What is a thread? **A:** The smallest unit of execution within a process, with its own stack and program counter.

**Q:** What creates the main thread in a Java program? **A:** The JVM, automatically, before `main()` runs.

**Q:** What's the difference between `start()` and `run()`? **A:** `start()` creates a new OS thread and runs the code concurrently; `run()` just executes the method on the current thread with no concurrency.

**Q:** What memory is shared between all threads? **A:** The heap.

**Q:** What memory is private to each thread? **A:** The stack (and program counter).

**Q:** Is thread execution order guaranteed in Java? **A:** No — it depends on the OS scheduler and is non-deterministic.

---

## References

- Oracle Java Tutorials — Concurrency: https://docs.oracle.com/javase/tutorial/essential/concurrency/
- Oracle Java Tutorials — Thread Objects: https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- `java.lang.Thread` API Documentation