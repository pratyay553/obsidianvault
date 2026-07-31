# Thread Creation

## Objective

By the end of this note, you will understand:

- The different ways to create a thread in Java
- The difference between extending `Thread` and implementing `Runnable`
- Which approach is preferred in real production code, and why
- How thread creation connects to the higher-level tools you'll learn later (`Callable`, `ExecutorService`)

This note builds on [[01 - What is Multithreading]] and [[02 - Process vs Thread]]. It's your first fully hands-on note in **Section 2: Creating & Running Threads**.

---

## Why should I learn this?

Before you can use any advanced concurrency tool in Java — thread pools, `CompletableFuture`, parallel streams — you need rock-solid fundamentals on how a thread is actually created and started. Almost every interview for a Java role will ask you to demonstrate at least one way of creating a thread, and many will ask you to explain the **trade-offs** between the two classic approaches.

Understanding this well also prevents a very common real bug: developers who don't understand the underlying mechanics often misuse thread creation in loops, leading to resource exhaustion in production (creating thousands of threads instead of reusing a pool).

---

## Beginner Explanation

There are two classic ways to tell Java "run this code on a new thread":

1. **Extend the `Thread` class** and override its `run()` method.
2. **Implement the `Runnable` interface**, write your logic inside `run()`, and pass that `Runnable` to a `Thread` object.

Think of it like giving instructions to a new employee (a thread):

- **Approach 1 (extend Thread)** is like literally writing the instructions directly on the employee's ID badge — the employee _is_ the instructions.
- **Approach 2 (implement Runnable)** is like writing the instructions on a separate piece of paper, and handing that paper to any available employee. The instructions (the task) are separate from the worker (the thread) carrying them out.

Approach 2 is almost always preferred in real-world Java code, and this note will explain exactly why.

---

## Real World Analogy

**Courier Service Analogy**

Imagine a courier service:

- **Extending Thread** is like hiring a courier who can _only_ ever deliver one specific, hardcoded package design — the courier and the package are fused together. If you want a different package delivered, you need an entirely new courier.
- **Implementing Runnable** is like having a pool of generic couriers, and handing any courier a package (the task) to deliver. The same courier can be reused for many different packages over time, and any courier can carry any package.

This is exactly why `Runnable` is preferred: it separates **the task** (what needs to be done) from **the worker** (the thread executing it), allowing tasks to be reused, queued, and assigned flexibly — especially important once you start using thread pools (see [[44 - Thread Pool]]).

---

## Technical Explanation

### Approach 1: Extending `Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}
```

You then create and start it:

```java
MyThread t = new MyThread();
t.start();
```

**Downsides:**

- Java does not support multiple inheritance — if your class extends `Thread`, it **cannot** extend any other class. This is a real limitation if your task logic naturally belongs to a class that already extends something else.
- It mixes "being a thread" with "doing a task," which violates the principle of separating responsibilities.

### Approach 2: Implementing `Runnable`

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}
```

You then wrap it in a `Thread` and start it:

```java
Thread t = new Thread(new MyTask());
t.start();
```

**Advantages:**

- Your task class can still extend another class, since `Runnable` is an interface, not a base class.
- The task (`Runnable`) is decoupled from the execution mechanism (`Thread`), which means the same `Runnable` can be submitted to a thread pool, executed directly, or scheduled — much more flexible.
- This is the approach expected in almost all real production Java code, especially once you move to `ExecutorService` (see [[46 - ExecutorService]]), which accepts `Runnable` (and `Callable`) tasks directly.

### Approach 3: Lambda Expressions (Modern Java, Java 8+)

Since `Runnable` is a **functional interface** (it has exactly one abstract method, `run()`), you can use a lambda expression instead of writing a full class:

```java
Thread t = new Thread(() -> {
    System.out.println("Running in: " + Thread.currentThread().getName());
});
t.start();
```

This is the most common style seen in modern Java code, tutorials, and interviews today, because it's concise and avoids boilerplate class definitions for simple tasks.

### Under the Hood: What `Thread` actually does with a `Runnable`

Internally, the `Thread` class itself implements `Runnable`. Its own `run()` method is defined roughly like this (simplified):

```java
public class Thread implements Runnable {
    private Runnable target;

    public void run() {
        if (target != null) {
            target.run();
        }
    }
}
```

So when you do `new Thread(myRunnable).start()`, the JVM creates a real OS thread, and when that new thread begins executing, it calls `Thread.run()`, which in turn calls `myRunnable.run()`. This is why extending `Thread` and overriding `run()` _also_ works — you're just overriding this exact delegation logic directly instead of supplying a separate `Runnable` object.

---

## Internal Working

Step-by-step of what happens when you create and start a thread using the `Runnable` approach:

1. You define a class (or lambda) implementing `Runnable`, with your logic inside `run()`.
2. You create a `Thread` object, passing your `Runnable` instance to its constructor. At this point, **no new thread exists yet** — this is just a regular Java object living on the heap.
3. You call `.start()` on the `Thread` object.
4. The JVM makes a native call to the OS, requesting a new OS-level thread.
5. The OS allocates a new call stack for this thread and registers it with the scheduler.
6. Once scheduled, the new thread begins executing the `Thread` object's internal `run()` method.
7. That internal `run()` method checks if a `target` (your `Runnable`) was provided — if so, it calls `target.run()`, which executes **your** logic.
8. When your `run()` method returns (finishes), the thread **terminates** — it cannot be restarted (calling `.start()` again throws `IllegalThreadStateException`).

---

## Memory Diagram

```
                Heap Memory
      -----------------------------
      |  Thread object (t)         |
      |    - target -> MyTask obj  |----+
      -----------------------------     |
                                          v
                                  ------------------
                                  | MyTask object   |
                                  |  (Runnable)     |
                                  ------------------

      When t.start() is called:

                OS creates a new native thread
                       |
                       v
              New Thread's Call Stack
      -----------------------------------
      | Thread.run() frame               |
      |   -> calls target.run()          |
      |      MyTask.run() frame          |
      -----------------------------------
```

---

## Execution Flow

```
main() thread
    |
    |-- creates Thread object t, wrapping a Runnable
    |-- calls t.start()
    |
    |------------------------------|
    |   main() continues        New OS thread created,
    |   its own work            begins executing
    |                           target.run() logic
    |------------------------------|
    |
    v                              v
main() finishes              New thread finishes
(independently, order not guaranteed)
```

---

## Code Example

```java
public class ThreadCreationDemo {

    // Approach 1: Extending Thread
    static class GreetingThread extends Thread {
        public GreetingThread(String name) {
            super(name); // sets the thread's name
        }

        @Override
        public void run() {
            System.out.println("[Extends Thread] Hello from " + Thread.currentThread().getName());
        }
    }

    // Approach 2: Implementing Runnable
    static class GreetingTask implements Runnable {
        @Override
        public void run() {
            System.out.println("[Implements Runnable] Hello from " + Thread.currentThread().getName());
        }
    }

    public static void main(String[] args) {

        // Approach 1
        GreetingThread t1 = new GreetingThread("Extend-Thread-Worker");
        t1.start();

        // Approach 2
        Thread t2 = new Thread(new GreetingTask(), "Runnable-Worker");
        t2.start();

        // Approach 3: Lambda (most common in modern code)
        Thread t3 = new Thread(() ->
                System.out.println("[Lambda] Hello from " + Thread.currentThread().getName()),
                "Lambda-Worker");
        t3.start();
    }
}
```

### Line-by-line explanation

1. `GreetingThread extends Thread` — this class **is** a thread; it overrides `run()` directly. Calling `super(name)` sets a custom thread name via `Thread`'s constructor.
2. `GreetingTask implements Runnable` — this class is just a **task**; it knows nothing about being a thread. It's handed to a `Thread` object separately.
3. `new Thread(new GreetingTask(), "Runnable-Worker")` — wraps the task in a `Thread`, giving it a name as the second constructor argument.
4. The lambda version (`t3`) shows the modern, concise style — since `Runnable` has a single abstract method, a lambda can be used directly instead of writing a full class.
5. All three approaches ultimately do the same thing at the OS level: request a new native thread and execute some `run()` logic on it. The differences are purely about **code organization and flexibility**, not runtime behavior.

---

## Dry Run

Given the code above, here's one possible (non-deterministic) output:

```
[Runnable-Worker] Hello from Runnable-Worker
[Extends Thread] Hello from Extend-Thread-Worker
[Lambda] Hello from Lambda-Worker
```

Or equally possible:

```
[Extends Thread] Hello from Extend-Thread-Worker
[Lambda] Hello from Lambda-Worker
[Runnable-Worker] Hello from Runnable-Worker
```

As always (see [[01 - What is Multithreading]]), the exact interleaving/order depends on the OS scheduler and is not guaranteed to be the same every run.

---

## Common Mistakes

1. **Extending `Thread` when you don't need to.**
    
    - If your class already extends another class, or if you just want to submit tasks to a thread pool later, `Runnable` is the correct choice. Overusing `Thread` inheritance is considered poor style in professional codebases.
2. **Calling `run()` directly instead of `start()`.**
    
    - As covered in [[01 - What is Multithreading]], this executes the code on the _current_ thread with zero concurrency benefit — a very common beginner mistake, repeated here because it's specific to thread creation and worth reinforcing.
3. **Reusing (restarting) a `Thread` object.**
    
    - Once a thread has finished running, calling `.start()` on that same object again throws `IllegalThreadStateException`. You must create a **new** `Thread` object if you want to run the task again (or better, use a thread pool that reuses worker threads internally — see [[44 - Thread Pool]]).
4. **Forgetting that `Runnable.run()` cannot return a value or throw checked exceptions.**
    
    - `Runnable`'s `run()` method has a `void` return type and cannot declare checked exceptions. If your task needs to return a result or throw a checked exception, you need `Callable` instead (see [[06 - Callable]]).
5. **Creating a new raw thread per task in a loop (e.g., inside a request handler).**
    
    - This does not scale — under load, this can create thousands of OS threads, exhausting memory and crippling performance. Production code should use `ExecutorService` and thread pools instead.

---

## Production Usage

- **Almost no production Java code manually extends `Thread` directly** for business logic — it's mostly seen in textbooks and interviews.
- **`Runnable` (and `Callable`)** are the standard way tasks are represented in production code, because they integrate directly with `ExecutorService`, `CompletableFuture`, and Spring's `@Async` mechanism.
- **Spring Boot's `@Async` methods** are internally executed as `Runnable`/`Callable` tasks submitted to a configured thread pool.
- **Web servers (Tomcat)** represent each incoming HTTP request internally as a task (essentially a `Runnable`) submitted to a worker thread pool — you almost never see raw `new Thread()` calls in this layer of well-designed frameworks.

---

## Interview Questions

**Beginner:**

1. What are the two classic ways to create a thread in Java?
2. What is the difference between extending `Thread` and implementing `Runnable`?
3. Can you use a lambda expression to create a thread? Why or why not?

**Intermediate:** 4. Why is implementing `Runnable` generally preferred over extending `Thread`? 5. What happens if you call `.start()` twice on the same `Thread` object? 6. Can a `Runnable`'s `run()` method return a value? If not, what should you use instead?

**Advanced:** 7. Internally, how does the `Thread` class use a `Runnable` object when you pass one into its constructor? 8. Why can't a class that extends `Thread` also extend another class in Java?

**Production-Level:** 9. Why do production systems almost always avoid manually creating raw `Thread` objects for business logic, and what do they use instead? 10. How does the `Runnable`/task separation from `Thread` (the worker) make thread pooling possible?

---

## Follow-up Questions

- If a `Thread` object is created but `.start()` is never called, does a new OS thread ever get created? _(No — a `Thread` object with `.start()` never called is just an inert Java object; no OS-level thread resources are consumed.)_
- Is `Runnable` a class or an interface? _(An interface — specifically, a functional interface with a single abstract method, `run()`.)_
- Can the same `Runnable` instance be passed to multiple different `Thread` objects? _(Yes — this is one of the key benefits of separating the task from the thread; the same task logic can run on multiple different threads, even concurrently, as long as the logic itself is safe to run concurrently.)_

---

## Best Practices

- Prefer implementing `Runnable` (or `Callable` when you need a return value) over extending `Thread`.
- Prefer lambda expressions for simple, short tasks — reserve full classes for more complex, reusable task logic.
- Never manually create large numbers of raw threads in a loop — always use a thread pool (`ExecutorService`) in production code (see [[44 - Thread Pool]], [[45 - Executor Framework]]).
- Always give threads meaningful names for easier debugging via thread dumps (see [[16 - Thread Naming]], [[69 - Thread Dump Analysis]]).

---

## Performance Notes

- Creating a `Thread` object itself is cheap (just a Java object allocation) — the expensive part is calling `.start()`, which triggers real OS-level thread creation (stack allocation, scheduler registration).
- Repeatedly creating and destroying threads (instead of reusing them via a pool) adds unnecessary OS overhead in high-throughput systems — this is precisely the problem thread pools solve.

---

## Comparison Table

|Approach|Can extend another class?|Reusable task across threads?|Common in production?|
|---|---|---|---|
|Extend `Thread`|No|No (task is fused to thread)|Rare|
|Implement `Runnable`|Yes|Yes|Very common|
|Lambda (`Runnable`)|Yes (implicitly)|Yes|Most common in modern code|
|`Callable` (returns value)|Yes|Yes|Common with `ExecutorService`|

---

## Summary

- Java offers three practical ways to define thread logic: extending `Thread`, implementing `Runnable`, and using lambda expressions (a shorthand for `Runnable`).
- Implementing `Runnable` is almost always preferred because it separates the task from the execution mechanism, allows your class to extend something else, and integrates cleanly with thread pools.
- `Thread` internally implements `Runnable` itself — extending `Thread` just means overriding this delegation logic directly.
- Production code rarely creates raw threads manually; it uses `ExecutorService` and thread pools instead, built on top of `Runnable`/`Callable`.

---

## Cheat Sheet

|Task|Code|
|---|---|
|Extend Thread|`class MyThread extends Thread { public void run() {...} }`|
|Implement Runnable|`class MyTask implements Runnable { public void run() {...} }`|
|Lambda Runnable|`new Thread(() -> { ... }).start();`|
|Start a thread|`thread.start();`|
|Common mistake|Calling `.run()` instead of `.start()`|

---

## Keywords

Thread, Runnable, Lambda, Functional Interface, Task, Extend, Implement, `start()`, `run()`, IllegalThreadStateException

---

## Related Notes

[[01 - What is Multithreading]] [[05 - Runnable]] [[06 - Callable]] [[44 - Thread Pool]] [[45 - Executor Framework]]

---

## Practice Exercises

**Easy:**

1. Create a thread by extending `Thread`, and print a custom message from it.
2. Create the same behavior using `Runnable` instead, and compare the two code structures.

**Medium:** 3. Write a `Runnable` task and pass the exact same instance to 3 different `Thread` objects. Start all 3 and observe that the same task logic runs on 3 different thread names. 4. Try calling `.start()` twice on the same `Thread` object and observe the exception that's thrown.

**Hard:** 5. Write a small program demonstrating why extending `Thread` becomes a problem when your class already needs to extend another class (e.g., try making a class extend both `Thread` and another custom class, and observe the compiler error).

---

## Mini Project

**Project: Task Runner Library (Mini Version)**

Build a small utility that:

1. Accepts any `Runnable` task and a thread name.
2. Wraps it in a `Thread`, starts it, and logs the start time, thread name, and end time once finished (use `.join()` to wait for completion).
3. Test it with 3 different tasks: a simple print statement, a `Thread.sleep`-based delay task, and a task that throws an exception (observe how uncaught exceptions in a thread behave).

**Goal:** Build comfort with the mechanics of thread creation, naming, and lifecycle before moving into thread pools.

---

## Common Interview Trap

**Trap Question:** _"Since `Thread` already implements `Runnable`, why would you ever bother implementing `Runnable` separately instead of just extending `Thread`?"_

The trap here is that beginners assume "fewer moving parts" (just extend `Thread`) is simpler and therefore better. The correct, deeper answer is about **design flexibility**: implementing `Runnable` keeps your task logic independent of the threading mechanism, allows your class to extend other classes (Java has no multiple inheritance), and — most importantly for real systems — allows the exact same task to be submitted to a thread pool via `ExecutorService`, rather than being permanently fused to one specific `Thread` object.

---

## Frequently Asked Questions

**Q: Can I pass a lambda directly where a `Thread` constructor expects a `Runnable`?** A: Yes — since `Runnable` is a functional interface (one abstract method), Java allows a lambda expression to be used anywhere a `Runnable` is expected.

**Q: If I implement `Runnable`, do I still need to call `new Thread(...)`?** A: Yes — implementing `Runnable` only defines the _task_. You still need a `Thread` object (or an `ExecutorService`) to actually execute it concurrently. Calling `.run()` directly on a `Runnable` alone just executes it like a normal method call, with no new thread involved.

**Q: Is there a performance difference between extending `Thread` and implementing `Runnable`?** A: No meaningful runtime performance difference — the difference is purely about code design and flexibility, not speed.

**Q: What happens if my `Runnable`'s `run()` method throws an unchecked exception?** A: The thread terminates, and the exception is passed to that thread's uncaught exception handler (by default, it prints a stack trace to `System.err`), but the JVM process and other threads generally continue running.

---

## Revision Checklist

- [ ] I can create a thread using both the `Thread`-extension approach and the `Runnable`-implementation approach.
- [ ] I understand why `Runnable` is preferred in production code.
- [ ] I understand that `Thread` internally implements `Runnable` and delegates to it.
- [ ] I know that a `Thread` object cannot be restarted once it has finished running.
- [ ] I understand the difference between `Runnable` (no return value) and `Callable` (returns a value) at a high level.

---

## Flashcards

**Q:** What are the three common ways to define thread logic in Java? **A:** Extending `Thread`, implementing `Runnable`, or using a lambda expression (shorthand for `Runnable`).

**Q:** Why is implementing `Runnable` generally preferred over extending `Thread`? **A:** It separates the task from the thread, avoids Java's single-inheritance limitation, and integrates with thread pools.
or
"I prefer implementing `Runnable` because it separates the task from the thread that executes it, making the code cleaner and more reusable. It also allows my class to extend another class since Java supports only single inheritance. Finally, `Runnable` works directly with the `ExecutorService`, which is the recommended way to manage threads in modern Java applications."

**Q:** Can you restart a `Thread` object after it has finished running? **A:** No — calling `.start()` again throws `IllegalThreadStateException`; you must create a new `Thread` object.

**Q:** What does `Thread` internally do with a `Runnable` passed to its constructor? **A:** It stores it as its `target` and calls `target.run()` from within its own `run()` method once the new thread starts executing.

**Q:** Does implementing `Runnable` by itself create a new thread? **A:** No — you still need to wrap it in a `Thread` object (or submit it to an `ExecutorService`) and call `.start()` (or submit it) to actually run it concurrently.

---

## References

- Oracle Java Tutorials — Defining and Starting a Thread: https://docs.oracle.com/javase/tutorial/essential/concurrency/threads.html
- `java.lang.Thread` API Documentation
- `java.lang.Runnable` API Documentation