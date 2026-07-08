# Process vs Thread

## Objective

By the end of this note, you will understand:

- What a **process** is, technically and intuitively
- What a **thread** is, and how it differs from a process
- How the JVM itself is a process, and how your Java threads live inside it
- Why this distinction matters for memory usage, isolation, and crash behavior

This note builds directly on [[01 - What is Multithreading]]. If you haven't read that yet, start there first.

---

## Why should I learn this?

Many beginners use the words "process" and "thread" interchangeably — but they are fundamentally different concepts, and confusing them leads to wrong assumptions about:

- Memory isolation (can one thread crash and take down the whole application? can one process?)
- Communication cost (why is talking between two threads cheap, but talking between two processes expensive?)
- Resource usage (why do processes take longer to start than threads?)

Understanding this distinction is essential because:

- Interviewers use this question to filter out candidates who only memorized definitions
- It explains **why** Java threads are lightweight compared to spinning up a new JVM process
- It is the foundation for understanding distributed systems, microservices, and containers later in your career

---

## Beginner Explanation

Think of your computer as an office building.

- A **process** is like an entire **department** in the building (e.g., the Sales Department). It has its own office space, its own filing cabinets, its own budget — completely separate from other departments.
- A **thread** is like an **employee** working inside that department. Multiple employees (threads) in the same department (process) share the same office space and filing cabinets (memory), but each employee still does their own individual work.

If the Sales Department's office burns down (the process crashes), every employee in it is affected. But if the Sales Department and the Marketing Department are separate, one department's problems don't necessarily affect the other.

In computing terms:

- A **process** = an independent running program with its own memory space.
- A **thread** = a unit of execution _inside_ that process, sharing the process's memory with other threads in the same process.

---

## Real World Analogy

**Bank Analogy**

- A **process** is like an entire **bank branch**. It has its own vault, its own staff, its own systems — fully independent from other branches.
- A **thread** is like a **teller** working inside that branch. Multiple tellers (threads) share the same vault and same customer records (shared memory), and can serve different customers at the same time.

If one teller makes a mistake, it might affect shared bank records used by other tellers (this is the concurrency risk we'll study in later notes). But if an entire branch (process) shuts down, none of the other branches are affected — they're fully isolated.

---

## Technical Explanation

### What is a Process?

A **process** is an independent, running instance of a program. Each process has its own:

- **Virtual memory address space** (isolated from other processes by the OS)
- **Heap, stack(s), and code segment**
- **File descriptors / open handles**
- **Environment variables**
- **At least one thread** (every process has a minimum of one thread — often called the "main thread" of that process)

The operating system enforces **strict isolation** between processes. One process cannot directly read or write another process's memory (unless using special OS-level inter-process communication, or IPC, mechanisms like pipes, sockets, or shared memory segments).

### What is a Thread?

A **thread** exists _inside_ a process. Threads within the same process:

- **Share** the process's heap memory, open file handles, and global/static variables
- **Do not share** their own call stack, program counter, or local variables — these are private per thread

### The JVM as a Process

When you run `java MyApp`, the operating system creates **one process** for the JVM. Inside that single process:

- The JVM creates a **main thread**
- Your code can create additional threads (worker threads, background threads, etc.)
- All these threads **share the same JVM heap memory** — meaning they can all access the same objects, static variables, and class metadata

This is why creating a new **thread** is much cheaper than creating a new **process** — a thread reuses the existing process's memory space, while a new process requires the OS to set up an entirely new, isolated memory space from scratch.

### Cost Comparison

|Operation|Approx. Cost|
|---|---|
|Creating a new **process**|Expensive — new memory space, new OS-level bookkeeping, slower (often milliseconds)|
|Creating a new **thread**|Cheaper — reuses existing process memory, faster (often microseconds), but still involves OS-level stack allocation|

---

## Internal Working

Here is what happens step by step when you start a Java application and it spawns additional threads:

1. You run `java MyApp` from the terminal.
2. The OS creates a **new process** for the JVM. This process gets its own isolated virtual address space.
3. The JVM initializes inside this process: it loads class files, sets up the heap, and starts the **main thread**.
4. Your `main()` method runs on this main thread.
5. If your code calls `new Thread(...).start()`, the JVM does **not** create a new process — it asks the OS to create a new **thread within the same process**.
6. This new thread gets:
    - Its own private stack (allocated by the OS)
    - Shared access to the same heap, class metadata, and static variables as every other thread in this process
7. If this JVM process crashes (e.g., `OutOfMemoryError` that terminates the JVM), **all threads within it die together** — because they all belonged to the same process.
8. If you wanted true isolation (so a crash in one part doesn't affect another), you would need to run **separate JVM processes** (e.g., separate microservices), not just separate threads.

---

## Memory Diagram

```
                     Operating System
       -------------------------------------------
       |                                          |
       |     Process A (JVM Instance #1)          |
       |   --------------------------------       |
       |   |         Heap Memory          |       |
       |   |   (shared within Process A)  |       |
       |   --------------------------------       |
       |     |         |         |               |
       |     v         v         v               |
       |  Thread-1  Thread-2  Thread-3            |
       |  (own      (own      (own                |
       |   stack)    stack)    stack)              |
       -------------------------------------------

       -------------------------------------------
       |                                          |
       |     Process B (JVM Instance #2)          |
       |   --------------------------------       |
       |   |         Heap Memory          |       |
       |   |  (COMPLETELY SEPARATE from   |       |
       |   |   Process A's heap)          |       |
       |   --------------------------------       |
       |     |                                    |
       |     v                                    |
       |  Thread-1 (own stack)                    |
       -------------------------------------------

     Process A and Process B CANNOT directly
     access each other's memory. Threads within
     the SAME process CAN share memory.
```

---

## Execution Flow

```
OS boots
   |
   v
You run: java MyApp
   |
   v
OS creates Process P  (isolated memory space)
   |
   v
JVM starts inside Process P
   |
   v
JVM creates Main Thread (runs main())
   |
   v
main() creates Thread T1, T2, T3
   |
   v
T1, T2, T3 run CONCURRENTLY, sharing
Process P's heap memory
   |
   v
If Process P crashes -> T1, T2, T3, and
Main Thread ALL die together
```

---

## Code Example

```java
public class ProcessVsThreadDemo {

    // A static variable lives in the PROCESS's heap memory,
    // shared by ALL threads inside this process.
    static int sharedCounter = 0;

    public static void main(String[] args) throws InterruptedException {

        System.out.println("This JVM instance is ONE process.");
        System.out.println("Main thread name: " + Thread.currentThread().getName());

        Runnable task = () -> {
            // Every thread can read/write sharedCounter
            // because it lives in the same process's heap.
            sharedCounter++;
            System.out.println(Thread.currentThread().getName() +
                    " incremented sharedCounter to " + sharedCounter);
        };

        Thread t1 = new Thread(task, "Worker-1");
        Thread t2 = new Thread(task, "Worker-2");

        t1.start();
        t2.start();

        t1.join(); // wait for t1 to finish (covered in [[20 - Join()]])
        t2.join(); // wait for t2 to finish

        System.out.println("Final sharedCounter value: " + sharedCounter);
    }
}
```

### Line-by-line explanation

1. `static int sharedCounter = 0;` — this variable lives in the **process's heap memory** (specifically, in the method area/metaspace for static fields). Every thread inside this one JVM process can see and modify it.
2. `Runnable task = () -> {...}` — defines the work each thread will perform.
3. `new Thread(task, "Worker-1")` — creates a new thread **within the same process**. Note it's given a name for clarity (see [[16 - Thread Naming]]).
4. `t1.start()` and `t2.start()` — both threads begin running concurrently, both accessing the **same** `sharedCounter` variable, because they belong to the **same process**.
5. `t1.join()` and `t2.join()` — the main thread waits for both worker threads to finish before printing the final value (without this, the print might happen before the workers finish).
6. Notice: **if this program were split into two separate Java processes** instead (e.g., `java MyApp` run twice), each process would have its **own independent `sharedCounter`**, starting at 0 each time — they would NOT share state, because separate processes do not share memory.

---

## Dry Run

Let's trace execution:

|Step|Thread|Action|sharedCounter value|
|---|---|---|---|
|1|Main|Starts t1, t2|0|
|2|Worker-1|Reads 0, increments, writes 1|1|
|3|Worker-2|Reads 1, increments, writes 2|2|
|4|Main|`join()` completes for both|2|
|5|Main|Prints "Final sharedCounter value: 2"|2|

**Important caveat:** the exact interleaving of steps 2 and 3 is not guaranteed — Worker-2 might run before Worker-1, or (without proper synchronization) they could even interleave in a way that causes a **race condition** where the final value ends up being 1 instead of 2 (both read 0 before either writes). We're not fixing that in this note — this is intentionally previewed here and solved in [[26 - Race Condition]] and [[22 - Synchronization]].

---

## Common Mistakes

1. **Assuming two separate Java program runs (`java MyApp` run twice) share memory.**
    
    - They do not. Each run creates a completely separate OS process with its own isolated heap. Static variables are **not** shared across separate JVM instances.
2. **Confusing "thread crash" with "process crash."**
    
    - If one thread throws an uncaught exception, by default only that thread dies (the exception propagates up to that thread's default uncaught exception handler) — the JVM process and other threads typically continue running. However, certain severe errors (like `OutOfMemoryError` in some cases) can bring down the entire process.
3. **Thinking threads are "mini processes."**
    
    - Threads are not isolated like processes — they intentionally share memory so they can cooperate efficiently. This shared memory is powerful but also the root cause of most concurrency bugs.
4. **Not realizing that spinning up new processes for parallel work (instead of threads) is much more expensive** in terms of memory and startup time — this matters when designing systems that need to scale.
    

---

## Production Usage

- **Microservices architecture**: Each microservice typically runs as its **own process** (often its own container), providing full isolation — if one microservice crashes, others are unaffected. Within each microservice, multiple **threads** handle concurrent requests efficiently.
- **Application servers (Tomcat, Spring Boot embedded server)**: Run as a single process, using many threads internally to handle concurrent HTTP requests — much cheaper than spinning up a new process per request.
- **Containers (Docker/Kubernetes)**: Each container typically runs one or more processes; scaling "horizontally" (more containers/processes) provides isolation, while scaling "vertically" within a process (more threads) is cheaper but shares fate if that process crashes.
- **Batch processing / ETL pipelines**: Often use multiple threads within a single JVM process to parallelize processing of large datasets, since threads are far cheaper than spinning up multiple JVM instances.

---

## Interview Questions

**Beginner:**

1. What is the difference between a process and a thread?
2. Does a process always have at least one thread?
3. Can two separate Java program runs share the same static variable value?

**Intermediate:** 4. Why is creating a new thread cheaper than creating a new process? 5. What happens to all threads in a process if that process crashes? 6. What memory do threads within the same process share, and what do they NOT share?

**Advanced:** 7. How does the OS provide memory isolation between processes, and why doesn't it provide the same isolation between threads in the same process? 8. What are some OS-level mechanisms for inter-process communication (IPC), and why are they more expensive than simple thread communication?

**Production-Level:** 9. In a microservices architecture, why might you choose separate processes (services) instead of just using more threads within one large process? 10. What are the trade-offs of using multiple threads within a single process versus scaling out to multiple processes/containers?

---

## Follow-up Questions

- If a thread throws an uncaught `RuntimeException`, does the whole JVM process die? _(Usually no — only that thread terminates, unless it was carrying critical state or the exception is something severe like `OutOfMemoryError` affecting the whole JVM.)_
- Can two threads in different JVM processes communicate directly through shared Java objects? _(No — they would need OS-level IPC like sockets, files, or shared memory segments, not direct object references.)_
- Why do containers (like Docker) still benefit from multithreading inside them, even though containers already provide process-level isolation? _(Because within a single container/process, you still want to utilize multiple CPU cores efficiently and handle multiple concurrent tasks — that's what threads are for.)_

---

## Best Practices

- Use **separate processes** (or containers) when you need strong fault isolation between independent components (e.g., separate microservices).
- Use **threads** within a single process when tasks need to share data efficiently and don't require full isolation.
- Don't rely on static variables to "share state" across separate JVM instances — that only works within a single process's threads.
- When designing systems, think in two dimensions: **scale up** (more threads, same process) vs **scale out** (more processes/machines) — and understand the trade-offs of each.

---

## Performance Notes

- **Thread creation** is generally 10x–100x cheaper (in time) than **process creation**, because the OS reuses the existing memory space.
- **Inter-thread communication** (sharing a variable) is essentially free (just memory access, though it needs proper synchronization for correctness) compared to **inter-process communication**, which requires OS-level mechanisms (sockets, pipes, shared memory) with real overhead.
- **Context switching between threads in the same process** is generally cheaper than **context switching between processes**, because process switches often require flushing more CPU cache and changing virtual memory mappings.

---

## Comparison Table

|Aspect|Process|Thread|
|---|---|---|
|Memory space|Own isolated virtual memory|Shares heap with other threads in same process|
|Creation cost|Expensive (new memory space)|Cheap (reuses process memory)|
|Communication|Requires IPC (sockets, pipes, shared memory)|Direct shared memory access|
|Crash impact|Isolated — doesn't affect other processes|A severe crash can bring down the whole process (and all its threads)|
|Context switch cost|Higher|Lower|
|Example|Separate `java MyApp` runs, separate microservices|Worker threads inside one JVM instance|

---

## Summary

- A **process** is an independent running program with its own isolated memory.
- A **thread** exists _inside_ a process and shares that process's memory with other threads.
- The JVM itself runs as **one process**; any threads you create in your Java code live inside that same process and share its heap.
- Threads are cheaper to create and communicate through than processes, but this shared memory is exactly what introduces concurrency bugs — the topic of much of the rest of this vault.
- Separate JVM runs = separate processes = separate, unshared memory.

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|Process|Independent program instance with isolated memory|
|Thread|Execution unit inside a process, sharing that process's memory|
|JVM instance|Runs as one OS process|
|Static variables|Shared across threads within the SAME process only|
|Thread creation cost|Cheap (reuses process memory)|
|Process creation cost|Expensive (new isolated memory space)|
|Crash impact (process)|Isolated from other processes|
|Crash impact (thread)|May be isolated, but severe cases can kill the whole process|

---

## Keywords

Process, Thread, JVM Instance, Isolation, Shared Memory, Heap, IPC (Inter-Process Communication), Virtual Address Space, Microservices, Context Switching

---

## Related Notes

[[01 - What is Multithreading]] [[03 - Why Multithreading Exists]] [[62 - Java Memory Model]] [[68 - JVM Thread Management]]

---

## Practice Exercises

**Easy:**

1. Write a program with a static `int` variable, and print its value from two different threads within the same program.
2. Run the same program (`java MyApp`) twice in two separate terminal windows, and confirm the static variable does NOT carry over between the two runs.

**Medium:** 3. Create a program with 3 threads, each incrementing a shared static counter 1000 times. Run it several times and observe if the final printed value is always exactly 3000 (spoiler: it might not be — this previews race conditions). 4. Research (or test) what happens if one thread in your program throws an uncaught `RuntimeException` — does the rest of the program keep running?

**Hard:** 5. Simulate "separate processes" behavior by writing two independent `main()` classes with the same static variable name, running them separately, and proving they don't affect each other. Then simulate "shared threads" behavior with a single class using multiple threads modifying one static variable — compare the two behaviors side by side.

---

## Mini Project

**Project: Process vs Thread Visualizer (Console-Based)**

Build a small console application that:

1. Prints the current process ID (`ProcessHandle.current().pid()` in modern Java).
2. Creates 3 threads, each printing their thread name and the same process ID.
3. Demonstrates that all 3 threads share the **same** process ID (proving they belong to the same process) while having different thread names (proving they're distinct threads).

**Goal:** Visually and practically confirm the process vs thread relationship, rather than just memorizing the definition.

---

## Common Interview Trap

**Trap Question:** _"If I run the same Java program twice at the same time, will they share the same static variables?"_

Many beginners incorrectly say "yes" because they conflate "same class/code" with "same running memory." The correct answer is **no** — each `java MyApp` invocation creates a **completely separate OS process** with its own isolated JVM heap. Static variables are only shared **within** a single process's threads, never across separate process instances, even if it's literally the exact same `.class` file being run.

---

## Frequently Asked Questions

**Q: Does every process need at least one thread?** A: Yes — a process with zero threads would have nothing executing any code; every process has at least one thread (often called the "main thread" of that process).

**Q: Can threads from two different processes directly share a Java object reference?** A: No. Object references are only meaningful within the memory space of a single process. Cross-process communication requires serialization and OS-level mechanisms (sockets, files, shared memory, message queues), not direct object sharing.

**Q: Is a Docker container the same thing as a process?** A: Not exactly — a container typically wraps one or more processes with additional isolation (namespaces, cgroups) provided by the OS, but at its core, the code running inside still executes as OS process(es) with their own threads.

**Q: Why don't operating systems let threads have full memory isolation like processes do?** A: Because the entire point of threads is to **share** memory efficiently for fast cooperation. Adding full isolation would eliminate the main performance benefit of using threads over processes.

---

## Revision Checklist

- [ ] I can clearly explain the difference between a process and a thread without confusing the two.
- [ ] I understand that the JVM itself runs as one process.
- [ ] I understand that static variables are shared across threads in the same process, but NOT across separate process instances.
- [ ] I understand why thread creation is cheaper than process creation.
- [ ] I can explain why microservices often use separate processes for isolation, while using threads internally for concurrency.

---

## Flashcards

**Q:** What is a process? **A:** An independent, running instance of a program with its own isolated memory space.

**Q:** What is a thread? **A:** A unit of execution inside a process that shares that process's memory with other threads.

**Q:** Do two separate JVM instances (two separate `java MyApp` runs) share static variables? **A:** No — each is a separate process with its own isolated memory.

**Q:** Which is cheaper to create — a thread or a process? **A:** A thread — it reuses the existing process's memory space instead of creating a new one.

**Q:** If a JVM process crashes entirely, what happens to all its threads? **A:** They all terminate together, since they all belonged to that one process.

---

## References

- Oracle Java Tutorials — Processes and Threads: https://docs.oracle.com/javase/tutorial/essential/concurrency/procthread.html
- `java.lang.ProcessHandle` API Documentation
- Operating Systems Concepts (Silberschatz et al.) — Process and Thread chapters (general OS reference)