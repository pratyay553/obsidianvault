---
tags: [concurrency]
---

# Concurrent Collections

## 🧠 Problem
SAloon's in-memory `Map<UUID, Slot>` of today's slots is read and written by multiple request-handling threads. A plain `HashMap` under concurrent modification can corrupt its internal structure or throw `ConcurrentModificationException`.

## 🤔 Why does the problem exist
Regular collections (`HashMap`, `ArrayList`) aren't designed for concurrent structural modification — internally they assume single-threaded access, and concurrent writes can corrupt internal pointers (infinite loops in old `HashMap` implementations are a known real-world bug class).

## 💡 Idea
Use collection types built for concurrent access: `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue` — each with different trade-offs for read/write frequency.

## 🏗️ Structure
```
ConcurrentHashMap        — segmented/striped locking, safe concurrent get/put
CopyOnWriteArrayList     — copies the array on every write; great for read-heavy, rarely-written lists
BlockingQueue            — thread-safe queue with blocking put/take, great for producer-consumer
```

## ☕ Java Implementation
```java
Map<UUID, Slot> todaysSlots = new ConcurrentHashMap<>();
todaysSlots.computeIfAbsent(stylistId, k -> new Slot()); // atomic check-then-insert

List<BookingObserver> observers = new CopyOnWriteArrayList<>(); // rarely mutated, read constantly

BlockingQueue<NotificationJob> notificationQueue = new LinkedBlockingQueue<>();
```

## 🏢 SAloon Example
`todaysSlots` is a `ConcurrentHashMap` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. The observer list in [[../05 - Design Patterns/Behavioral/Observer]] is a `CopyOnWriteArrayList` since subscriptions rarely change but notifications fire constantly.

## 🎯 When should I use it?
Any shared collection accessed by multiple threads — pick the specific type based on read/write ratio (read-heavy → `CopyOnWriteArrayList`; balanced → `ConcurrentHashMap`; producer/consumer → `BlockingQueue`).

## ⚠️ When NOT to use it?
`CopyOnWriteArrayList` copies the entire backing array on every write — terrible choice for write-heavy lists. Match the collection type to the actual access pattern.

## 🔥 Interview Question
*"Why not just wrap a HashMap in `Collections.synchronizedMap()`?"*
→ That locks the *entire map* for every operation (coarse-grained), while `ConcurrentHashMap` uses finer-grained internal locking, giving much better throughput under concurrent access.

## 🔗 Related
[[00 - Thread Safety]]
[[02 - Locks]]

#concurrency
