# 70 - Debugging Multithreading

Tags: #jvm-internals #section12 #debugging
Related: [[69 - Thread Dump Analysis]] | [[71 - Profiling Threads]] | [[73 - Common Production Bugs]] | [[36 - Deadlock]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Learn a systematic approach to debugging concurrent Java applications — from reproducing the bug to finding the root cause.

---

## Beginner Explanation
Concurrency bugs are hard because they're non-deterministic — they appear intermittently, often disappear under a debugger, and may depend on exact thread scheduling. You need a systematic toolkit rather than trial-and-error.

---

## Technical Deep Dive

### Why concurrency bugs are hard
- **Non-deterministic**: scheduling depends on OS and CPU state
- **Heisenbug**: adding `System.out.println` or pausing in a debugger changes timing, making the bug disappear
- **No stack trace**: a race condition doesn't throw an exception — it silently corrupts data

### Debugging toolkit

#### 1. Thread Dumps (first tool to reach for) → [[69 - Thread Dump Analysis]]
- Deadlock: `jstack <pid>` → look for "Found one Java-level deadlock"
- Starvation: repeatedly dump every 10s — some threads never in RUNNABLE
- High CPU: find RUNNABLE threads with the same hot stack frame across dumps

#### 2. Logging — ThreadName in every log line
```java
// Logback pattern with thread name:
// %d{HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
log.info("Processing order {}", orderId);
// → 14:23:01 [http-nio-8080-exec-3] INFO  OrderService - Processing order 123
```
Add a correlation ID (request ID, trace ID) that flows across threads → use `MDC`:
```java
MDC.put("requestId", requestId);
// all log lines in this thread now include the request ID
```

#### 3. Java Flight Recorder (JFR) — best for production profiling
```bash
jcmd <pid> JFR.start name=debug duration=60s filename=debug.jfr
# Then analyze in JMC (Java Mission Control)
```
JFR captures: thread states, lock contention, CPU usage, GC, allocations — very low overhead (~1%).

#### 4. VisualVM / JConsole (GUI tools)
- Monitor live thread count, CPU, heap
- Thread tab: visual state of each thread
- Thread dump button: capture on demand

#### 5. Instrumentation / StampedLock / custom debugging
```java
// Temporarily add lock contention logging
ReentrantLock lock = new ReentrantLock() {
    @Override
    public void lock() {
        if (isLocked() && !isHeldByCurrentThread()) {
            log.warn("Thread {} waiting for lock held by {}",
                Thread.currentThread().getName(), "...");
        }
        super.lock();
    }
};
```

#### 6. `-ea` assertions for invariant checking
```java
assert Thread.holdsLock(this) : "Must hold lock before calling this method";
```

#### 7. FindBugs / SpotBugs / SonarQube
Static analysis that detects common concurrency bugs:
- Unsynchronized access to shared fields
- Double-checked locking errors
- `wait()` not in a loop
- `notify()` instead of `notifyAll()`

### Systematic debugging process
1. **Reproduce**: write a stress test or load test that reliably triggers the bug
2. **Isolate**: binary search — remove code until bug disappears to find minimum reproduction
3. **Classify**: is it a deadlock, race condition, visibility bug, or starvation?
4. **Instrument**: add logging/JFR without changing behavior
5. **Hypothesize and verify**: form a theory, write a test that proves/disproves it
6. **Fix and prevent**: fix the root cause; add a regression test

### Common fixes by bug class
| Bug | Symptom | Fix |
|---|---|---|
| Race condition | Corrupted data intermittently | Synchronize or use atomic classes |
| Deadlock | Hang, all threads BLOCKED | Lock ordering, `tryLock` with timeout |
| Visibility | Stale reads, infinite loops | `volatile` or synchronization |
| Starvation | Some threads never progress | Fair locks, priority adjustment |
| Thread leak | Thread count grows forever | Proper `ExecutorService.shutdown()` |

---

## Code

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.*;

public class RaceConditionDebug {
    // BAD — race condition
    static int counter = 0;
    static void badIncrement() { counter++; } // read-modify-write not atomic

    // GOOD — atomic
    static AtomicInteger atomicCounter = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        int threads = 100, iterations = 1000;

        // Demonstrate the race
        ExecutorService pool = Executors.newFixedThreadPool(threads);
        CountDownLatch latch = new CountDownLatch(threads);
        for (int i = 0; i < threads; i++) {
            pool.submit(() -> {
                for (int j = 0; j < iterations; j++) {
                    badIncrement();         // racy
                    atomicCounter.incrementAndGet(); // safe
                }
                latch.countDown();
            });
        }
        latch.await();
        pool.shutdown();

        System.out.println("Expected: " + (threads * iterations));
        System.out.println("counter (racy):  " + counter); // < expected
        System.out.println("atomic (correct): " + atomicCounter.get()); // == expected
    }
}
```

---

## Interview Questions

**Q: Why do concurrency bugs often disappear when you add a debugger?**
> Debuggers pause threads and add synchronization points that alter scheduling. Breakpoints introduce artificial happens-before relations that mask races.

**Q: What is the best first step when diagnosing a production hang?**
> Take 2-3 thread dumps with `jstack <pid>` spaced 10-30 seconds apart. Look for threads stuck in the same state across all dumps — deadlocks, blocked threads, or threads in infinite loops.

**Q: How do you detect a race condition without seeing corrupted data yet?**
> Static analysis (SpotBugs), stress testing with many threads, or code review looking for unsynchronized compound operations (`check-then-act`, `read-modify-write`).

---

## Cheat Sheet
| Tool | Best for |
|---|---|
| `jstack` | Deadlocks, BLOCKED threads |
| JFR | CPU, lock contention, allocation profiling |
| VisualVM | Live thread monitoring |
| SpotBugs | Static race condition detection |
| Logging + MDC | Request tracing across threads |
| Stress test | Reproducing race conditions |
