# Thread Pool

## Objective

By the end of this note, you will understand:

- Why creating a new raw thread per task doesn't scale, precisely
- What a thread pool is and how it solves this problem
- The core lifecycle of tasks flowing through a pool: submission, queuing, execution, completion
- The key configuration parameters that determine a pool's behavior under load

This note opens **Section 8: Thread Pools & Executors** — the single most practically important section for everyday production Java work, since almost nobody creates raw threads directly in real systems.

---

## Why should I learn this?

Recall from [[01 - What is Multithreading]] and [[04 - Thread Creation]]: creating a raw `Thread` has real OS-level cost (memory for its stack, scheduler registration), and creating thousands of them for short-lived tasks is a fast path to resource exhaustion. A **thread pool** solves this by maintaining a **reusable** set of worker threads that process a continuous stream of tasks — this is, without exaggeration, one of the most universally used patterns in production Java, underlying web servers, database connection handling, async processing, and more.

---

## Beginner Explanation

Imagine a restaurant that, instead of hiring a brand-new chef for every single customer order (and firing them the moment the dish is done), keeps a **fixed team of chefs** on staff. Orders come in continuously and get placed in a queue; whenever a chef finishes their current dish, they pick up the next order from the queue. The restaurant never needs more chefs than seats at the stove, and chefs are **reused** across many orders throughout their shift, rather than being a one-time, disposable resource per order.

A thread pool works exactly this way: a fixed (or bounded) set of worker threads, continuously pulling tasks from a shared queue and executing them, rather than a new thread being created and destroyed for every single task.

---

## Real World Analogy

**Warehouse Analogy**

Imagine a warehouse with a fixed team of 10 workers and a conveyor belt bringing in a continuous stream of packages to process. Rather than hiring a brand-new worker for every single package (and letting them go immediately after), the warehouse keeps its 10 workers on staff, and each grabs the next package off the belt as soon as they finish their current one. If packages arrive faster than the 10 workers can process them, they simply **queue up** on the belt, waiting their turn — nobody is turned away, they just wait a bit longer during busy periods.

This "fixed team, continuous queue, reuse workers" model is precisely how a thread pool operates.

---

## Technical Explanation

### The Core Components of a Thread Pool

1. **A fixed (or bounded) set of worker threads** — created once, reused across many tasks.
2. **A task queue** (typically a `BlockingQueue`, see [[43 - BlockingQueue]]) — holds tasks waiting for an available worker thread.
3. **A submission mechanism** — how new tasks (`Runnable`/`Callable`) get added to the queue.
4. **A lifecycle** — the pool itself can be running, shutting down, or terminated.

### Why Raw Thread Creation Doesn't Scale

Recall from [[12 - Context Switching]] and [[01 - What is Multithreading]]:

- Each raw thread consumes real memory for its stack (~512KB-1MB by default).
- Creating/destroying threads has real OS-level overhead.
- Far more threads than CPU cores leads to excessive context switching.

A web server handling 10,000 requests per second, if it created a new raw thread per request, would rapidly exhaust memory and cripple performance through context-switching overhead — this is precisely the scenario thread pools are designed to prevent.

### The Basic Flow

```
Task submitted --> placed in queue (if all workers busy)
                --> OR immediately picked up (if a worker is free)
Worker thread --> pulls task from queue --> executes it --> returns to pool, ready for next task
```

### Simplified Conceptual Implementation

```java
class SimpleThreadPool {
    private final BlockingQueue<Runnable> taskQueue = new LinkedBlockingQueue<>();
    private final List<Thread> workers = new ArrayList<>();

    SimpleThreadPool(int numThreads) {
        for (int i = 0; i < numThreads; i++) {
            Thread worker = new Thread(() -> {
                while (true) {
                    try {
                        Runnable task = taskQueue.take(); // blocks until a task is available
                        task.run();
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                        return;
                    }
                }
            }, "PoolWorker-" + i);
            worker.start();
            workers.add(worker);
        }
    }

    void submit(Runnable task) {
        taskQueue.add(task);
    }
}
```

This is, in essence, exactly what `ThreadPoolExecutor` (the real JDK implementation, fully detailed in [[46 - ExecutorService]]) does internally — a fixed set of long-lived worker threads, each looping forever, pulling tasks from a shared `BlockingQueue`.

### Key Configuration Concepts

- **Core pool size**: the number of threads kept alive even when idle.
- **Maximum pool size**: the upper limit the pool can grow to under heavy load (some pool types allow growing beyond the core size temporarily).
- **Queue capacity**: how many pending tasks can wait before the pool must either grow (if allowed) or reject new submissions.
- **Keep-alive time**: how long "extra" threads (beyond the core size) stay alive while idle before being terminated.

These parameters are covered in full depth in [[46 - ExecutorService]] and [[72 - Performance Tuning]] — this note focuses on the conceptual foundation they all build on.

---

## Internal Working

Step-by-step of a task's journey through a thread pool:

1. Application code calls `submit(task)` (or `execute(task)`).
2. The pool checks: is a worker thread currently idle and available?
3. **If yes**: the task is handed directly to that worker, which begins executing it immediately.
4. **If no** (all workers busy): the task is placed into the internal task queue, waiting its turn.
5. As soon as any worker finishes its current task, it pulls the next task from the queue (if any) and begins executing it.
6. This cycle repeats indefinitely — workers are **never destroyed** after a single task; they simply loop, continuously processing whatever comes next.

---

## Memory Diagram

```
        Task Queue (BlockingQueue)
   -----------------------------------
   |  [Task4, Task5, Task6, ...]       |
   -----------------------------------
            |
   ---------+---------+---------
   |                  |         |
   v                  v         v
 Worker-1          Worker-2  Worker-3
 (executing        (executing (idle,
  Task1)            Task2)     just took
                               Task3)

   Workers loop FOREVER: finish task -> take next from
   queue -> execute -> repeat (never destroyed per-task)
```

---

## Execution Flow

```
submit(task)
        |
        v
Is a worker idle right now?
        |
   -----------------------------
   |                           |
  YES                         NO
   |                           |
   v                           v
Hand directly to        Add to task queue,
that worker              wait for a worker
   |                     to become free
   v                           |
Worker executes task            |
   |                           |
   v                           v
Worker becomes idle again, checks queue for next task
        |
        v
(Cycle repeats -- worker threads are REUSED, never
 destroyed after a single task)
```

---

## Code Example

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.List;
import java.util.ArrayList;

public class ThreadPoolDemo {

    static class SimplePool {
        private final BlockingQueue<Runnable> taskQueue = new LinkedBlockingQueue<>();
        private final List<Thread> workers = new ArrayList<>();
        private volatile boolean running = true;

        SimplePool(int numThreads) {
            for (int i = 0; i < numThreads; i++) {
                Thread worker = new Thread(() -> {
                    while (running) {
                        try {
                            Runnable task = taskQueue.take();
                            task.run();
                        } catch (InterruptedException e) {
                            Thread.currentThread().interrupt();
                            return;
                        }
                    }
                }, "PoolWorker-" + i);
                worker.start();
                workers.add(worker);
            }
        }

        void submit(Runnable task) {
            taskQueue.add(task);
        }

        void shutdown() throws InterruptedException {
            running = false;
            for (Thread w : workers) w.interrupt();
            for (Thread w : workers) w.join();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        SimplePool pool = new SimplePool(3); // only 3 worker threads

        for (int i = 1; i <= 10; i++) {
            int taskId = i;
            pool.submit(() -> {
                System.out.println(Thread.currentThread().getName() + " executing task " + taskId);
                try { Thread.sleep(300); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            });
        }

        Thread.sleep(3500); // let tasks finish
        pool.shutdown();
        System.out.println("Pool shut down.");
    }
}
```

### Line-by-line explanation

1. `SimplePool` creates exactly 3 worker threads, each running an infinite loop that pulls tasks from `taskQueue` and executes them.
2. `submit()` simply adds a task to the shared queue — it does **not** create a new thread; the existing 3 workers handle everything.
3. Submitting 10 tasks to a pool of 3 workers means tasks queue up and get processed **in batches of 3 at a time**, with each worker being **reused** across multiple tasks (e.g., `PoolWorker-0` might execute tasks 1, 4, 7, 10, cycling back for more as it finishes each one).
4. This directly demonstrates the core value proposition: 10 tasks were handled by just 3 threads, reused continuously, rather than 10 separate, disposable threads.

---

## Dry Run

Example output (exact task-to-worker assignment varies):

```
PoolWorker-0 executing task 1
PoolWorker-1 executing task 2
PoolWorker-2 executing task 3
PoolWorker-0 executing task 4
PoolWorker-1 executing task 5
PoolWorker-2 executing task 6
PoolWorker-0 executing task 7
PoolWorker-1 executing task 8
PoolWorker-2 executing task 9
PoolWorker-0 executing task 10
Pool shut down.
```

Notice: only 3 distinct worker thread names appear across all 10 tasks — direct, visible proof of reuse.

---

## Common Mistakes

1. **Creating a new raw `Thread` for every task in production code.**
    
    - This is precisely the anti-pattern thread pools exist to solve — under any meaningful load, this leads to resource exhaustion (see [[01 - What is Multithreading]]'s "Common Mistakes" section for the original warning this note now fully explains).
2. **Sizing a pool without considering whether the workload is CPU-bound or I/O-bound.**
    
    - As covered in [[03 - Why Multithreading Exists]] and [[12 - Context Switching]], CPU-bound work benefits from a pool size near the core count, while I/O-bound work can often support many more threads — this distinction directly informs thread pool sizing decisions (fully detailed in [[72 - Performance Tuning]]).
3. **Using an unbounded task queue without considering memory risk.**
    
    - Just like with `BlockingQueue` in general (see [[43 - BlockingQueue]]), an unbounded queue removes backpressure — if tasks are submitted faster than the pool can process them indefinitely, memory usage can grow without bound.
4. **Forgetting to properly shut down a thread pool.**
    
    - As covered in [[13 - Daemon Thread]]/[[14 - User Thread]], pool worker threads are typically non-daemon by default — forgetting to shut down a pool can prevent the JVM from exiting.

---

## Production Usage

- **Web servers (Tomcat, Jetty)**: use thread pools to handle incoming HTTP requests, with a bounded number of worker threads processing a continuous stream of connections.
- **Database connection pools**: conceptually related — a fixed set of reusable connections rather than creating a new one per query, following the same "reuse expensive resources" philosophy as thread pools.
- **Async task processing in Spring Boot** (`@Async`, `TaskExecutor`): built directly on top of Java's thread pool abstractions.
- **Virtually every non-trivial concurrent Java application** uses thread pools somewhere, whether directly via `ExecutorService` (see [[46 - ExecutorService]]) or indirectly through a framework that manages one internally.

---

## Interview Questions

**Beginner:**

1. What is a thread pool, and what problem does it solve?
2. Why is creating a new thread for every task considered bad practice at scale?
3. What are the two main components of a basic thread pool (workers and what else)?

**Intermediate:** 4. What happens to a task when all worker threads in a pool are currently busy? 5. Are worker threads destroyed after completing a task? 6. Why might an unbounded task queue be risky in production?

**Advanced:** 7. Explain, step by step, how a task submitted to a pool eventually gets executed, from submission to completion. 8. How does the appropriate thread pool size differ between CPU-bound and I/O-bound workloads?

**Production-Level:** 9. Name two real-world systems (besides a custom application) that rely on thread pools internally. 10. What operational risk exists if a thread pool is never properly shut down?

---

## Follow-up Questions

- Can a thread pool grow or shrink dynamically based on load? _(Depends on configuration — some pool types have a fixed size; others can grow up to a maximum under load and shrink back down during idle periods, covered fully in [[46 - ExecutorService]]'s different factory methods.)_
- Does submitting a task to a full pool's queue block the submitting thread? _(Depends on the queue implementation and submission method — some configurations block, others reject the task immediately or apply a specific rejection policy, covered in [[46 - ExecutorService]].)_
- Is a thread pool the same thing as a `BlockingQueue`? _(No — a `BlockingQueue` is one **component** typically used internally by a thread pool to hold pending tasks; the pool itself also manages the worker threads, their lifecycle, and the overall submission/execution flow.)_

---

## Best Practices

- Always use a thread pool (via `ExecutorService`, see [[46 - ExecutorService]]) rather than creating raw threads for repeated or high-volume task execution.
- Size your pool deliberately based on whether your workload is CPU-bound or I/O-bound.
- Prefer a bounded task queue in most production scenarios, to provide backpressure and avoid unbounded memory growth.
- Always ensure your application properly shuts down any thread pools it creates, typically during application shutdown.

---

## Performance Notes

- Thread pools amortize the cost of thread creation across many tasks — creating threads once and reusing them dramatically reduces the overhead compared to per-task thread creation.
- An appropriately-sized pool avoids the excessive context-switching overhead (see [[12 - Context Switching]]) that would result from having far more threads than useful parallelism for the workload type.
- Queue capacity and pool size together determine a system's behavior under burst load — this is a genuine, important tuning consideration, not a "set and forget" decision.

---

## Comparison Table

|Aspect|Raw Thread Per Task|Thread Pool|
|---|---|---|
|Thread creation cost|Paid for every single task|Paid once, amortized across many tasks|
|Resource usage under high volume|Can exhaust memory/OS resources|Bounded, predictable|
|Task queuing under load|None — tasks just spawn more threads|Tasks queue, processed as workers free up|
|Typical production usage|Rare, discouraged|Standard, expected practice|

---

## Summary

- A thread pool maintains a fixed (or bounded) set of reusable worker threads that continuously process tasks from a shared queue, avoiding the cost and risk of creating a new raw thread per task.
- Tasks are submitted, queued if all workers are busy, and picked up by the next available worker — which then loops back for more work rather than being destroyed.
- Proper sizing depends on workload type (CPU-bound vs. I/O-bound) and expected load patterns.
- Thread pools underlie web servers, async frameworks, and essentially all serious concurrent Java applications — this is foundational, everyday production knowledge.
- The JDK's real implementation, `ThreadPoolExecutor` (accessed via `ExecutorService`), is covered in full depth in [[46 - ExecutorService]].

---

## Cheat Sheet

|Concept|One-liner|
|---|---|
|Thread pool|Fixed/bounded set of reusable worker threads|
|Task queue|Holds pending tasks when all workers are busy|
|Worker lifecycle|Loops forever, never destroyed per-task|
|CPU-bound sizing|~ number of cores|
|I/O-bound sizing|Can exceed core count significantly|

---

## Keywords

Thread Pool, Worker Thread, Task Queue, Reuse, ThreadPoolExecutor, Backpressure, Bounded Queue

---

## Related Notes

[[01 - What is Multithreading]] [[12 - Context Switching]] [[43 - BlockingQueue]] [[45 - Executor Framework]] [[46 - ExecutorService]] [[72 - Performance Tuning]]

---

## Practice Exercises

**Easy:**

1. Reproduce this note's `SimplePool` demo, and confirm that only as many distinct worker thread names appear as you configured.
2. Reduce the pool size to 1 and observe that all 10 tasks now execute strictly sequentially, one after another.

**Medium:** 3. Modify `SimplePool` to print a message whenever a worker picks up a NEW task versus when it's idle waiting, to visualize the reuse pattern more explicitly. 4. Experiment with submitting tasks of varying durations (some fast, some slow) and observe how the queue naturally load-balances work across the fixed worker set.

**Hard:** 5. Extend `SimplePool` to support a bounded queue with a simple rejection policy (e.g., throw an exception if the queue is full when a new task is submitted), and test this behavior under a burst of submissions exceeding the queue's capacity.

---

## Mini Project

**Project: Thread Pool From Scratch**

Build a more complete version of `SimplePool` that:

1. Supports a configurable core pool size and a bounded task queue.
2. Implements a basic rejection policy (e.g., caller-runs, where the submitting thread itself executes the task if the queue is full) for when the queue is full.
3. Provides a proper `shutdown()` method that stops accepting new tasks but lets already-queued tasks finish before terminating worker threads.
4. Includes a test harness submitting a large burst of tasks and confirming correct behavior under load.

**Goal:** Build a genuinely functional, simplified thread pool from first principles — directly preparing you to deeply understand `ThreadPoolExecutor`'s real, more sophisticated implementation in [[46 - ExecutorService]].

---

## Common Interview Trap

**Trap Question:** _"If a thread pool has 5 worker threads, and I submit 5 long-running tasks followed immediately by 1 more quick task, will the quick task get a 'fair chance' to run before the long tasks finish?"_

The trap is assuming some kind of fairness or task-based prioritization exists by default. The correct answer: **no** — a standard thread pool's task queue is (by default, in most common configurations) simple FIFO ordering with **no** awareness of task duration or any concept of "fairness" beyond arrival order. If all 5 workers are immediately occupied by the 5 long-running tasks, the 6th (quick) task will simply **wait in the queue** until one of the 5 workers finishes its current long-running task — regardless of how "quick" the waiting task actually is. If you need priority-aware scheduling, you'd need to use a `PriorityBlockingQueue` (see [[43 - BlockingQueue]]) as the pool's task queue instead of the default FIFO queue — this isn't automatic, and must be a deliberate configuration choice.

---

## Frequently Asked Questions

**Q: Is a thread pool's size fixed forever once created?** A: Not necessarily — depends on configuration; some pool types (like `Executors.newCachedThreadPool()`, see [[46 - ExecutorService]]) can grow and shrink dynamically based on load, while others (like `Executors.newFixedThreadPool()`) maintain a constant size.

**Q: Can a thread pool have zero worker threads temporarily?** A: Yes, for pool types that allow core threads to time out when idle (a configurable option) — though a "fixed" pool by default keeps its core threads alive even when idle.

**Q: Does a thread pool guarantee tasks execute in the exact order they were submitted?** A: Generally yes, for a standard FIFO queue, **if** the pool has more workers than concurrently-submitted tasks or tasks are submitted with enough spacing; but with multiple workers processing concurrently, tasks can genuinely finish out of submission order even if they started being processed in order, since execution durations can vary.

**Q: Is it possible for all worker threads in a pool to be busy while the task queue is also empty?** A: Yes — this simply means the pool is at full utilization with no backlog; new tasks submitted at this moment would need to wait for a worker to free up (or be queued, depending on configuration).

---

## Revision Checklist

- [ ] I understand why creating a raw thread per task doesn't scale.
- [ ] I can explain the core flow: submission, queuing (if needed), execution, worker reuse.
- [ ] I understand that worker threads are never destroyed after a single task.
- [ ] I know the basic sizing considerations for CPU-bound versus I/O-bound workloads.
- [ ] I understand the risk of an unbounded task queue.

---

## Flashcards

**Q:** What is the core purpose of a thread pool? **A:** To reuse a fixed/bounded set of worker threads across many tasks, avoiding the cost of creating a new thread per task.

**Q:** What happens to a task if all workers are currently busy? **A:** It waits in the task queue until a worker becomes free.

**Q:** Are worker threads destroyed after completing a task? **A:** No — they loop back to pick up the next available task.

**Q:** What data structure typically holds pending tasks in a thread pool? **A:** A `BlockingQueue`.

**Q:** Why is an unbounded task queue potentially risky? **A:** It removes backpressure, potentially allowing unbounded memory growth if tasks are submitted faster than they can be processed.

---

## References

- Oracle Java Tutorials — Executors: https://docs.oracle.com/javase/tutorial/essential/concurrency/executors.html
- `java.util.concurrent.ThreadPoolExecutor` API Documentation
- `java.util.concurrent.BlockingQueue` API Documentation