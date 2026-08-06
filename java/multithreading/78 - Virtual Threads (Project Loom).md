# 78 - Virtual Threads (Project Loom)

Tags: #frameworks #section13 #java21
Related: [[02 - Process vs Thread]] | [[12 - Context Switching]] | [[75 - Tomcat Thread Pool]] | [[79 - Structured Concurrency]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Master Virtual Threads (Project Loom, Java 21) — the biggest threading change in Java's history that makes one-thread-per-task scalable for millions of concurrent I/O-bound tasks.

---

## Beginner Explanation
Before Java 21, every Java thread was a real OS thread — expensive (~1MB stack, slow to create). Virtual threads are **JVM-managed lightweight threads** that run on top of a small pool of real OS threads. You can have millions of them, and they're cheap to create and block.

---

## Analogy
Platform threads are like taxis (one driver per car, limited fleet). Virtual threads are like Uber — thousands of "virtual" drivers backed by a much smaller pool of actual cars. When a virtual driver is waiting (blocked on I/O), the car is available for another driver.

---

## Technical Deep Dive

### Platform thread vs Virtual thread

| Aspect | Platform Thread | Virtual Thread |
|---|---|---|
| Maps to | 1 OS thread | Multiplexed on carrier threads |
| Stack size | ~512KB–2MB (fixed) | ~1KB (grows on demand) |
| Creation cost | High | Near-zero |
| Max practical count | ~thousands | ~millions |
| Blocking cost | Blocks OS thread | Unmounts from carrier |
| Scheduling | OS scheduler | JVM scheduler (ForkJoinPool) |
| Java version | All | Java 21+ (GA) |

### How virtual threads work
1. Virtual thread is **mounted** onto a carrier (OS) thread (ForkJoinPool worker)
2. It runs until it **blocks** (e.g., `socket.read()`, `Thread.sleep()`, blocking DB call)
3. The JVM **unmounts** the virtual thread from the carrier — carrier is free for other virtual threads
4. When the blocking operation completes, the virtual thread is **remounted** on a (possibly different) carrier

This is transparent — your code looks identical to platform thread code.

### Creating virtual threads

```java
// Java 21 API:
Thread vt = Thread.ofVirtual().start(() -> handleRequest());
Thread vt = Thread.ofVirtual().name("vt-1").unstarted(task);

// Factory:
ThreadFactory factory = Thread.ofVirtual().factory();
Thread t = factory.newThread(task);

// ExecutorService — one virtual thread per task:
ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor();
exec.submit(() -> handleRequest()); // millions of these is fine
```

### When virtual threads shine
✅ I/O-bound workloads (HTTP calls, DB queries, file I/O) — thread blocks → JVM parks it cheaply  
✅ High-concurrency servers (thousands of simultaneous requests)  
✅ Migration of existing thread-per-request code — drop-in replacement  

### When virtual threads don't help
❌ CPU-bound tasks — virtual threads don't create more CPU cores  
❌ Code that pins the carrier thread (native methods, `synchronized` on long operations)  

### Pinning — the main gotcha
When a virtual thread executes `synchronized` code that blocks, it **pins** the carrier thread (the OS thread doesn't unmount). Many pinned virtual threads = carrier thread starvation.

```java
// PINS carrier — bad for long blocking inside synchronized
synchronized (obj) {
    socket.read(); // virtual thread pins the OS carrier thread here
}

// FIX — use ReentrantLock instead
lock.lock();
try {
    socket.read(); // virtual thread unmounts cleanly, carrier is free
} finally {
    lock.unlock();
}
```

Detect pinning: JVM flag `-Djdk.tracePinnedThreads=full`

### Virtual threads and ThreadLocal
With millions of virtual threads, `ThreadLocal` per thread = massive memory explosion. Java 21 introduces **Scoped Values** (`ScopedValue`) as the preferred alternative:

```java
// Scoped Values — Java 21+
static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

ScopedValue.where(REQUEST_ID, "req-42").run(() -> {
    System.out.println(REQUEST_ID.get()); // "req-42"
    // child virtual threads also see this value
});
```

### Spring Boot 3.2+ with virtual threads
```yaml
spring:
  threads:
    virtual:
      enabled: true  # Tomcat + @Async use virtual threads automatically
```

---

## Code

```java
import java.util.concurrent.*;

public class VirtualThreadDemo {
    public static void main(String[] args) throws Exception {
        // 1. Basic virtual thread
        Thread.ofVirtual().start(() ->
            System.out.println("Virtual thread: " + Thread.currentThread().isVirtual())
        ).join();

        // 2. One virtual thread per task executor
        try (ExecutorService exec = Executors.newVirtualThreadPerTaskExecutor()) {
            // Submit 10,000 tasks — fine with virtual threads, not with platform threads
            List<Future<Integer>> futures = new ArrayList<>();
            for (int i = 0; i < 10_000; i++) {
                final int id = i;
                futures.add(exec.submit(() -> {
                    Thread.sleep(100); // each blocks 100ms — platform threads would saturate
                    return id;
                }));
            }
            long sum = 0;
            for (Future<Integer> f : futures) sum += f.get();
            System.out.println("Sum: " + sum); // 49,995,000
        }

        // 3. Check pinning (bad pattern)
        Object lock = new Object();
        Thread.ofVirtual().start(() -> {
            synchronized (lock) {
                try { Thread.sleep(1000); } // pins carrier thread during sleep!
                catch (InterruptedException e) {}
            }
        }).join();

        // 4. Correct pattern — ReentrantLock
        ReentrantLock rl = new ReentrantLock();
        Thread.ofVirtual().start(() -> {
            rl.lock();
            try { Thread.sleep(1000); } // virtual thread unmounts, carrier is free
            catch (InterruptedException e) {}
            finally { rl.unlock(); }
        }).join();
    }
}
```

---

## Interview Questions

**Q: What is a virtual thread in Java 21?**
> A JVM-managed lightweight thread multiplexed onto a small pool of OS (carrier) threads. Cheap to create (~1KB stack), can have millions simultaneously. When blocked on I/O, the virtual thread unmounts from its carrier, freeing the OS thread for other virtual threads.

**Q: What is "pinning" in virtual threads?**
> When a virtual thread executes `synchronized` code that blocks, it holds its carrier OS thread — preventing unmounting. Many pinned threads starve the carrier pool. Fix: replace `synchronized` with `ReentrantLock` for blocking sections.

**Q: When should you NOT use virtual threads?**
> CPU-bound tasks — virtual threads don't add CPU cores; use a `ForkJoinPool` sized to core count. Also avoid when heavy use of `ThreadLocal` would create per-thread copies for millions of threads.

**Q: What is the difference between virtual threads and async/reactive programming?**
> Both address I/O scalability. Async/reactive (CompletableFuture, WebFlux) requires callback-based code that is hard to read/debug. Virtual threads allow **synchronous-looking blocking code** with the same scalability — simpler to write and debug.

---

## Cheat Sheet
| Concept | Detail |
|---|---|
| Create | `Thread.ofVirtual().start(task)` |
| Pool | `Executors.newVirtualThreadPerTaskExecutor()` |
| I/O blocking | Unmounts carrier — no waste |
| CPU-bound | No benefit — no extra cores |
| Pinning | `synchronized` + blocking = carrier stuck |
| Fix pinning | Use `ReentrantLock` instead |
| ThreadLocal alt | `ScopedValue` (Java 21+) |
| Spring Boot | `spring.threads.virtual.enabled=true` |
