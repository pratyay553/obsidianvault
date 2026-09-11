---
tags: [concurrency]
---

# Thread Safety

## 🧠 Problem
Two customers hit "book this slot" at nearly the same instant. Both threads read "slot is free," both proceed to book it. Now it's double-booked.

## 🤔 Why does the problem exist
Multiple threads reading and writing shared state (the slot's availability) without coordination can interleave in ways that break the "only one booking per slot" invariant — the check-then-act sequence isn't atomic.

## 💡 Idea
A piece of code/data structure is thread-safe if it behaves correctly no matter how multiple threads' operations interleave. Achieved via synchronization, atomics, immutability, or confinement (each thread has its own copy).

## 🏗️ Structure
```
Thread A: check slot free → [context switch] → book slot
Thread B:                    check slot free → book slot   ← both see "free"!
```

## ☕ Java Implementation (the fix)
```java
class SlotLockManager {
    private final Object lock = new Object();
    boolean tryBook(Slot slot) {
        synchronized (lock) {
            if (slot.isBooked()) return false;
            slot.markBooked(); // check-then-act is now atomic
            return true;
        }
    }
}
```

## 🏢 SAloon Example
`SlotLockManager.tryBook()` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. This exact bug (double-booking under load) is what forced me to add this lock; it didn't show up until concurrent load testing.

## 🎯 When should I use it?
Any shared mutable state touched by more than one thread — booking slots, counters, caches, connection pools.

## ⚠️ When NOT to use it?
Don't add synchronization to state that's actually thread-confined (only ever touched by one thread, or truly immutable) — unnecessary locking just adds contention and slows things down for no safety benefit.

## 🔥 Interview Question
*"Why doesn't just checking `if (!slot.isBooked())` before booking fix it?"*
→ Because "check" and "book" are two separate operations — a thread can be paused by the OS scheduler *between* them, letting another thread interleave. Only wrapping both in one atomic/synchronized unit closes that gap.

## 🔗 Related
[[01 - synchronized]]
[[05 - Race Conditions]]

#concurrency
