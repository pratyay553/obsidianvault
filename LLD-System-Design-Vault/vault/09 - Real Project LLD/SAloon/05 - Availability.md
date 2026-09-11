---
tags: [saloon, real-project]
---

# SAloon — Availability Module

## 🧠 Problem
Given a stylist's working hours and existing bookings, quickly determine which time slots are free — checked on almost every page view (browsing available times), so it needs to be fast, but also correct under concurrent bookings.

## 🤔 Why does the problem exist
Checking availability by scanning all of a stylist's bookings for every request doesn't scale once a popular stylist has hundreds of bookings, and a naive cache risks serving stale "available" data right as another customer books the same slot.

## 💡 Idea
Keep an in-memory `ConcurrentHashMap`-backed cache of today's slots, guarded by a `ReadWriteLock` since checks (reads) vastly outnumber actual bookings (writes). Started as a plain class (no pattern at all) and only introduced `Strategy` once staff-specific buffer-time rules were a real requirement — see [[../../03 - Design Principles/01 - KISS]].

## 🏗️ Structure
```
AvailabilityChecker
  - cache: ConcurrentHashMap<StylistId, List<Slot>>
  - lock: ReadWriteLock
  + isFree(slot): boolean          ← read lock
  + markBooked(slot)                ← write lock

AvailabilityRule (interface, added later) → DefaultRule, BufferTimeRule
```

## ☕ Java Implementation
```java
class AvailabilityChecker {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Map<UUID, List<Slot>> bookedSlots = new ConcurrentHashMap<>();

    boolean isFree(Slot slot) {
        lock.readLock().lock();
        try { return bookedSlots.getOrDefault(slot.getStylistId(), List.of())
                     .stream().noneMatch(b -> b.overlaps(slot)); }
        finally { lock.readLock().unlock(); }
    }

    void markBooked(Slot slot) {
        lock.writeLock().lock();
        try { bookedSlots.computeIfAbsent(slot.getStylistId(), k -> new ArrayList<>()).add(slot); }
        finally { lock.writeLock().unlock(); }
    }
}
```

## Why I resisted patterns here for months
This is the [[../../03 - Design Principles/01 - KISS|KISS]] example in this vault — `AvailabilityChecker` stayed a plain class with no `Strategy` until buffer-time rules (stylist needs 15 min between appointments) actually shipped as a requirement. Building the extension point earlier would've been speculative.

## Every pattern/principle this module uses
- [[../../07 - Concurrency/02 - Locks]] — `ReadWriteLock` for read-heavy access
- [[../../07 - Concurrency/04 - Concurrent Collections]] — `ConcurrentHashMap`
- [[../../03 - Design Principles/01 - KISS]] / [[../../03 - Design Principles/02 - YAGNI]] — deliberately delayed abstraction

## 🔥 Interview Question
*"Why ReadWriteLock instead of a plain synchronized method here, specifically?"*
→ Availability checks (reads) vastly outnumber actual bookings (writes) — a plain `synchronized` would serialize all the frequent reads unnecessarily. `ReadWriteLock` lets reads run concurrently and only serializes the rarer writes.

## 🔗 Related
[[00 - Booking]]
[[../../07 - Concurrency/02 - Locks]]

#saloon
