# 90 - Final Interview Revision Guide

Tags: #interview #section15
Related: [[81 - Interview Questions]] | [[83 - Revision Notes]] | [[85 - Cheat Sheet]] | [[84 - Glossary]]
Back to: [[00 - Java Multithreading Home]]

---

## How to Use This Note
This is your last stop before a Java multithreading interview. Read it the morning of. Every line is a trigger for a deeper topic — follow the WikiLinks to review anything that feels shaky.

---

## The Three Axes of Thread Safety

Every concurrency answer lives on three axes. Mention all three to sound senior:

| Axis | Question it answers | Tool |
|---|---|---|
| **Visibility** | Can Thread B see Thread A's write? | `volatile`, `synchronized` |
| **Atomicity** | Is the operation indivisible? | `AtomicXxx`, `synchronized` |
| **Ordering** | Can the compiler/CPU reorder operations? | `volatile`, `synchronized`, HB rules |

---

## Must-Know Definitions (15 seconds each)

- **Thread** — smallest execution unit; shares heap; own stack + program counter
- **Race condition** — outcome depends on thread scheduling; shared mutable state without sync
- **Deadlock** — circular lock dependency; neither thread can proceed
- **Livelock** — threads keep reacting to each other; no progress; high CPU
- **Starvation** — thread never gets CPU/lock because others always win
- **Happens-before** — formal guarantee: write by X is visible to Y if X HB Y
- **Volatile** — visibility + ordering; NOT atomicity for compound ops
- **CAS** — hardware compare-and-swap; basis of all lock-free structures
- **Monitor** — intrinsic lock on every Java object; used by `synchronized`
- **Virtual thread** — JVM-managed lightweight thread (Java 21); millions possible; unmounts on I/O

---

## State Machine — Draw This If Asked

```
         start()
NEW ──────────────▶ RUNNABLE ◀──────────────────────────┐
                      │  │                               │
         lock free    │  │ wants lock         notify()   │
                      │  ▼                               │
                   BLOCKED ──── lock acquired ──────────▶│
                      │                                  │
         wait()       │                                  │
    ──────────────▶ WAITING ──── notify() ──────────────▶│
                      │                                  │
    sleep/wait(ms) ▶TIMED_WAITING ── timeout/notify ────▶│
                      │
                      ▼
                  TERMINATED
```

---

## The 5 Rules That Prevent 90% of Bugs

1. **Never share mutable state without synchronization** — if two threads touch a field and one writes, you need `volatile`, `synchronized`, or `AtomicXxx`.
2. **Always call `remove()` in thread pools** — `ThreadLocal` data lives as long as the thread lives; pool threads live forever.
3. **Always check wait conditions in a `while` loop** — spurious wakeups are real; `if (queue.isEmpty()) wait()` is always wrong.
4. **Always acquire locks in a consistent global order** — two locks → define which comes first; enforce everywhere.
5. **Always call `finally { lock.unlock(); }`** — `ReentrantLock` doesn't auto-release on exception.

---

## Synchronization Tool Selection

```
Need to protect a single counter?
  → AtomicInteger / AtomicLong

High contention counter (many threads)?
  → LongAdder

Single field read/write visibility?
  → volatile

Compound operation (check-then-act, read-modify-write)?
  → synchronized block or ReentrantLock

Multiple wait conditions (producers AND consumers)?
  → ReentrantLock + two Conditions

Read-heavy, write-rare map?
  → ConcurrentHashMap (reads lock-free)

Read-heavy, write-rare list?
  → CopyOnWriteArrayList

Read-heavy, write-rare single object?
  → volatile reference + immutable object (swap atomically)

Multiple readers OR one writer?
  → ReadWriteLock

Optimistic reads (retry on write conflict)?
  → StampedLock

N tasks waiting for M events?
  → CountDownLatch (one-shot) or Phaser (reusable)

N threads meeting at a barrier?
  → CyclicBarrier

Bounded resource access?
  → Semaphore

Two threads swapping data?
  → Exchanger
```

---

## Thread Pool Selection

```
CPU-bound (cores tasks, no blocking)?
  → ForkJoinPool sized to CPU count

I/O-bound, Java < 21?
  → FixedThreadPool(N), N = cores × (1 + wait/service)

I/O-bound, Java 21+?
  → newVirtualThreadPerTaskExecutor()

Fire-and-forget tasks?
  → CachedThreadPool (careful: unbounded)

Recurring/scheduled?
  → ScheduledExecutorService

Async pipeline?
  → CompletableFuture with custom executor
```

---

## Interview Story Template

When asked "tell me about a concurrency problem you solved":

> **Situation**: [service] was experiencing [symptom] under [load conditions].  
> **Root cause**: [shared mutable state] was accessed by [N threads] without synchronization — specifically, [the exact operation that was unsafe].  
> **Impact**: [data loss / hang / OOM / timeout].  
> **Fix**: Replaced [broken primitive] with [correct primitive] because [reason].  
> **Prevention**: Added [test / monitoring / code review checklist].

See [[82 - Production Case Studies]] for 6 ready-made stories.

---

## Behavioural Questions — What They're Really Testing

| They ask... | They want to know... |
|---|---|
| "Describe a deadlock you fixed" | You can read thread dumps; you understand lock ordering |
| "How do you size a thread pool?" | You know it's not just "more = better"; CPU vs I/O distinction |
| "What's the difference between volatile and synchronized?" | Visibility vs. atomicity; compound ops |
| "How would you test thread-safe code?" | Stress tests, `@RepeatedTest`, `Thread.sleep` bad; probabilistic bugs |
| "Have you used reactive programming?" | You can compare reactive vs virtual threads trade-offs |

---

## Rapid-Fire Q&A Drill

**Q: Does `sleep()` release the lock?**  
A: ❌ No. `wait()` releases. `sleep()` does not.

**Q: Can `wait()` be called outside `synchronized`?**  
A: ❌ No — throws `IllegalMonitorStateException`.

**Q: What happens if you call `notify()` before `wait()`?**  
A: The signal is lost. The waiting thread waits forever. Use `CountDownLatch` when signal can arrive first.

**Q: Is `++` on an `int` field atomic?**  
A: ❌ No — read + increment + write. Use `AtomicInteger.incrementAndGet()`.

**Q: Is a `volatile long` write atomic?**  
A: ✅ Yes — `volatile` guarantees single-write atomicity for `long`/`double`.

**Q: Can `ConcurrentHashMap` store null keys?**  
A: ❌ No — `NullPointerException`. `HashMap` can.

**Q: What does `@Async` do in Spring?**  
A: Runs the method on a separate thread from the configured `TaskExecutor`. Only works on `public` methods called from outside the bean.

**Q: What is pinning in virtual threads?**  
A: When a virtual thread blocks inside `synchronized`, it cannot unmount from its carrier OS thread. Replace `synchronized` with `ReentrantLock`.

**Q: What is the difference between `ShutdownOnFailure` and `ShutdownOnSuccess`?**  
A: `ShutdownOnFailure` cancels all tasks if any fails. `ShutdownOnSuccess` cancels remaining when the first one succeeds.

**Q: When does `ForkJoinPool` outperform `FixedThreadPool`?**  
A: Recursive divide-and-conquer tasks where subtasks vary in size — work-stealing balances uneven load automatically.

---

## Common Trap Answers (What Juniors Say vs What Seniors Say)

| Topic | Junior says | Senior says |
|---|---|---|
| Race condition fix | "Use synchronized everywhere" | "Identify the smallest critical section; prefer atomics for simple counters; synchronize compound ops" |
| Thread pool size | "Use 100 threads" | "CPU-bound: N=cores. I/O-bound: measure wait/service ratio. Profile under real load." |
| `volatile` use | "Use it to make compound ops atomic" | "Volatile only guarantees visibility and ordering. For read-modify-write, use AtomicInteger or synchronized." |
| Deadlock | "Just use more locks" | "Deadlock requires circular dependency. Prevention: consistent lock ordering, tryLock with timeout." |
| Memory leak | "Restart the service" | "Thread dump + heap dump. Classic pattern: ThreadLocal not removed in pool threads. Add remove() in finally." |
| Virtual threads | "Just a faster thread" | "JVM-multiplexed onto carrier OS threads. Unmounts on blocking I/O. Cheap to create (millions). Avoid synchronized+blocking (pinning)." |

---

## Last 10-Minute Checklist

Before you walk in, confirm you can explain without notes:

- [ ] 6 thread states and how to reach each
- [ ] `sleep` vs `wait` (lock release, needs `synchronized`)
- [ ] `volatile` — what it does and does NOT do
- [ ] `synchronized` — reentrant, releases on exception, block vs method
- [ ] `AtomicInteger.compareAndSet()` — how CAS works
- [ ] `ReentrantLock` — `tryLock`, `Condition`, fair mode
- [ ] `CountDownLatch` vs `CyclicBarrier`
- [ ] `ConcurrentHashMap` vs `Collections.synchronizedMap`
- [ ] `ExecutorService` — `submit`, `shutdown`, `awaitTermination`
- [ ] `CompletableFuture` — `thenApply`, `thenCompose`, `allOf`, `exceptionally`
- [ ] Deadlock — definition + how to prevent
- [ ] ThreadLocal — purpose + the `remove()` rule
- [ ] Happens-before — at least 3 rules
- [ ] False sharing — cause + fix
- [ ] Virtual threads — what they are, pinning, Java 21

---

## Final Mindset

Concurrency bugs are **non-deterministic** — they appear under load, on specific hardware, or after days of uptime. The interviewer knows this. They're not testing whether you've memorised every API; they're testing whether you:

1. **Think systematically** about shared mutable state
2. **Choose appropriate tools** (not "synchronized everything")
3. **Know how to diagnose** from symptoms (thread dump, JFR, metrics)
4. **Have production instincts** (ThreadLocal leaks, pool exhaustion, deadlock patterns)

You've studied this vault cover to cover. You are ready.

---

*"Make it work. Make it correct. Make it fast. In that order."*

---

## Vault Map (Quick Nav)
| Section | Notes |
|---|---|
| Foundations | [[01]] – [[06]] |
| Thread Creation | [[07]] – [[11]] |
| Lifecycle | [[12]] – [[18]] |
| Thread Control | [[19]] – [[21]] |
| Synchronization | [[22]] – [[31]] |
| Advanced Locks | [[32]] – [[35]] |
| Classic Problems | [[36]] – [[39]] |
| Communication | [[40]] – [[43]] |
| Thread Pools | [[44]] – [[51]] |
| Concurrent Collections | [[52]] – [[61]] |
| JMM & Memory | [[62]] – [[66]] |
| JVM Internals | [[67]] – [[73]] |
| Frameworks | [[74]] – [[80]] |
| Interview Prep | [[81]] – [[85]] |
| Practice | [[86]] – [[90]] |
