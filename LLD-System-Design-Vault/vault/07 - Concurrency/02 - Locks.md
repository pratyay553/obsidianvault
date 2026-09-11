---
tags: [concurrency]
---

# Locks (ReentrantLock, ReadWriteLock)

## 🧠 Problem
`synchronized` is simple but rigid — I can't try to acquire a lock with a timeout, and I can't let multiple readers of "is this slot available" run concurrently while still blocking writers.

## 🤔 Why does the problem exist
`synchronized` blocks are all-or-nothing: a thread waits indefinitely, and readers block each other even though reading is safe to do concurrently.

## 💡 Idea
`java.util.concurrent.locks` gives finer control: `ReentrantLock` supports `tryLock(timeout)` and explicit `lock()`/`unlock()`; `ReadWriteLock` lets many readers proceed together but gives writers exclusive access.

## 🏗️ Structure
```
ReadWriteLock
  ├── readLock()   — many threads can hold this simultaneously
  └── writeLock()  — exclusive, blocks all readers and writers
```

## ☕ Java Implementation
```java
class AvailabilityCache {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private Map<UUID, Boolean> cache = new HashMap<>();

    boolean isAvailable(UUID slotId) {
        lock.readLock().lock();
        try { return cache.getOrDefault(slotId, true); }
        finally { lock.readLock().unlock(); }
    }

    void updateAvailability(UUID slotId, boolean available) {
        lock.writeLock().lock();
        try { cache.put(slotId, available); }
        finally { lock.writeLock().unlock(); }
    }
}
```

## 🏢 SAloon Example
The in-memory availability cache in front of the DB uses `ReadWriteLock` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. Availability checks (frequent, read-heavy) don't block each other; only actual booking writes take the exclusive lock.

## 🎯 When should I use it?
Read-heavy/write-light shared data (caches, config), or when you need timeout/interruptible locking that `synchronized` can't offer.

## ⚠️ When NOT to use it?
`ReentrantLock` requires manual `unlock()` in a `finally` block — forgetting it causes a permanent deadlock. If you don't need the extra features, `synchronized` is safer by default (auto-released).

## 🔥 Interview Question
*"Why always `unlock()` inside `finally`?"*
→ If an exception is thrown inside the locked section and `unlock()` is skipped, every future thread trying to acquire that lock blocks forever — `finally` guarantees release regardless of how the block exits.

## 🔗 Related
[[01 - synchronized]]
[[03 - Atomic Variables]]

#concurrency
