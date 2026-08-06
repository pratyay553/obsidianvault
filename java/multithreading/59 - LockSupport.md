# 59 - LockSupport

Tags: #coordination-utilities #section9 #advanced
Related: [[32 - ReentrantLock]] | [[35 - Condition]] | [[10 - Thread States]]
Back to: [[00 - Java Multithreading Home]]

---

## Objective
Understand `LockSupport` — the low-level primitive that underpins all Java concurrency locks and conditions.

---

## Beginner Explanation
`LockSupport` provides `park()` and `unpark(thread)` — the lowest-level thread blocking/unblocking mechanism in Java. `ReentrantLock`, `Semaphore`, `CountDownLatch`, and `Condition` all use `LockSupport` internally. You rarely call it directly unless building your own synchronizer.

---

## Analogy
`LockSupport` is the raw on/off switch for threads. Higher-level tools (`ReentrantLock`, etc.) are appliances built on top of that switch. You don't usually flip the switch directly — you use the appliances.

---

## Technical Deep Dive

### API
```java
LockSupport.park();                     // suspend current thread (indefinitely)
LockSupport.park(blocker);              // suspend with a "blocker" object for diagnostics
LockSupport.parkNanos(nanos);           // park up to N nanoseconds
LockSupport.parkUntil(deadline);        // park until absolute epoch time
LockSupport.unpark(thread);             // unblock target thread
LockSupport.getBlocker(thread);         // get the blocker object (for thread dumps)
```

### Permit model
Every thread has a **single binary permit** (0 or 1):
- `park()` — if permit is 1, consume it and return immediately (no block). If 0, block.
- `unpark(t)` — set thread t's permit to 1. If t is parked, it wakes up. If t is not parked, the next `park()` call returns immediately.

This means **`unpark()` can be called before `park()`** and the park won't block — unlike `wait()`/`notify()` where order matters.

### State in thread dump
A parked thread appears as `WAITING (parking)`:
```
"worker-1" prio=5
   java.lang.Thread.State: WAITING (parking)
   at sun.misc.Unsafe.park(Native Method)
   at java.util.concurrent.locks.LockSupport.park(LockSupport.java:...)
```

### Key differences from `wait()`/`notify()`
| Aspect | `wait`/`notify` | `LockSupport.park`/`unpark` |
|---|---|---|
| Requires lock | ✅ Yes (`synchronized`) | ❌ No |
| Order matters | ✅ Yes (notify before wait = lost) | ❌ No (unpark before park = ok) |
| Spurious wakeup | Possible | Possible |
| Target specific thread | ❌ No | ✅ Yes — `unpark(thread)` |

---

## Code

```java
import java.util.concurrent.locks.*;

public class LockSupportDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            System.out.println("Worker: about to park");
            LockSupport.park(); // blocks here
            System.out.println("Worker: unparked and running again");

            // Parkable with blocker for diagnostics
            LockSupport.park(Thread.currentThread()); // blocker shows in jstack
            System.out.println("Worker: second park done");
        });

        worker.start();
        Thread.sleep(500);

        System.out.println("Main: unparking worker");
        LockSupport.unpark(worker); // wake it up

        Thread.sleep(100);
        LockSupport.unpark(worker); // second unpark
        worker.join();

        // Demonstrate unpark-before-park
        Thread t2 = new Thread(() -> {
            try { Thread.sleep(200); } catch (InterruptedException e) {}
            System.out.println("T2: parking (already has permit)");
            LockSupport.park(); // returns immediately because permit already set
            System.out.println("T2: park returned instantly");
        });
        t2.start();
        LockSupport.unpark(t2); // set permit before t2 even parks
        t2.join();
    }
}
```

---

## Interview Questions

**Q: What is `LockSupport` and why does it exist?**
> `LockSupport` provides `park()`/`unpark()` — the lowest-level thread suspension primitives in Java. It is the building block for all `java.util.concurrent` locks and conditions. Unlike `wait`/`notify`, it doesn't require a monitor, and `unpark` can safely be called before `park`.

**Q: What happens if you call `unpark(thread)` before the thread calls `park()`?**
> The thread's permit is pre-set to 1. When it calls `park()`, it returns immediately without blocking.

**Q: How does `LockSupport` differ from `Object.wait()`?**
> `wait()` requires entering a `synchronized` block; `park()` does not. `unpark(thread)` targets a specific thread; `notify()` is non-deterministic. Permits in `park`/`unpark` are ordered-agnostic; `notify` before `wait` loses the signal.

---

## Cheat Sheet
| Method | Description |
|---|---|
| `park()` | Suspend current thread |
| `park(blocker)` | Suspend + attach diagnostic object |
| `parkNanos(n)` | Park up to N nanoseconds |
| `unpark(t)` | Unblock thread t |
| Requires lock | ❌ No |
| Pre-notification | ✅ Yes — unpark before park is safe |
