---
tags: [concurrency]
---

# synchronized keyword

## 🧠 Problem
`SlotLockManager.tryBook()` needs to be atomic across threads — no two threads should execute the check-and-book sequence at the same time for the same slot.

## 🤔 Why does the problem exist
Without any locking mechanism, the JVM is free to interleave two threads' instructions inside the same method arbitrarily.

## 💡 Idea
`synchronized` acquires a lock (monitor) on an object before entering the block/method, and releases it on exit — only one thread can hold that lock at a time, so the guarded code runs atomically with respect to other synchronized blocks on the *same* lock object.

## 🏗️ Structure
```
synchronized(lockObject) {
    // only one thread here at a time, for this lockObject
}
```

## ☕ Java Implementation
```java
class SlotLockManager {
    private final Map<UUID, Object> slotLocks = new ConcurrentHashMap<>();

    boolean tryBook(Slot slot) {
        Object lock = slotLocks.computeIfAbsent(slot.getId(), k -> new Object());
        synchronized (lock) { // per-slot lock, not one global lock
            if (slot.isBooked()) return false;
            slot.markBooked();
            return true;
        }
    }
}
```
Locking **per slot** (not one global lock for all slots) means bookings for *different* slots don't block each other — this granularity choice matters a lot for throughput.

## 🏢 SAloon Example
See [[../09 - Real Project LLD/SAloon/00 - Booking]] — per-slot locking is exactly why booking Slot A and Slot B for two different customers can happen fully in parallel.

## 🎯 When should I use it?
Guarding a critical section of shared mutable state — but prefer the smallest lock scope and finest-grained lock object that still guarantees correctness.

## ⚠️ When NOT to use it?
A single global `synchronized` on the whole class/method when only a small part touches shared state — creates a bottleneck. Also avoid it for read-heavy, rarely-written data (prefer `ReadWriteLock` — see [[02 - Locks]]).

## 🔥 Interview Question
*"What's the risk of over-synchronizing?"*
→ Reduced throughput (threads queue up even when they don't need to conflict) and potential deadlock if multiple locks are acquired in inconsistent order across different code paths.

## 🔗 Related
[[00 - Thread Safety]]
[[02 - Locks]]

#concurrency
