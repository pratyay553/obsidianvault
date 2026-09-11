---
tags: [concurrency]
---

# Atomic Variables

## 🧠 Problem
SAloon tracks a simple `activeBookingsCount` for a live dashboard. Using `synchronized` for a single counter increment feels like overkill — a full lock/unlock cycle for one `count++`.

## 🤔 Why does the problem exist
`count++` is actually three separate operations (read, increment, write) — under concurrent access without protection, two threads can read the same value and both write back the same incremented result, losing one increment (a classic "lost update").

## 💡 Idea
`AtomicInteger`/`AtomicLong` (and friends) use low-level CPU compare-and-swap (CAS) instructions to make single-variable read-modify-write operations atomic *without* taking a traditional lock — faster for simple counters under contention.

## 🏗️ Structure
```
AtomicInteger.incrementAndGet()
   → CPU-level compare-and-swap loop, retries if another thread changed it first
```

## ☕ Java Implementation
```java
class BookingMetrics {
    private final AtomicInteger activeBookingsCount = new AtomicInteger(0);
    void onBookingConfirmed() { activeBookingsCount.incrementAndGet(); }
    void onBookingCancelled() { activeBookingsCount.decrementAndGet(); }
    int getCount() { return activeBookingsCount.get(); }
}
```

## 🏢 SAloon Example
`BookingMetrics.activeBookingsCount` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. No `synchronized` block needed for this single counter, unlike the slot-booking logic which genuinely needs a lock across multiple fields.

## 🎯 When should I use it?
A single variable (counter, flag, reference) that needs atomic read-modify-write, with no other state that needs to change consistently alongside it.

## ⚠️ When NOT to use it?
The moment you need to update *two or more* related fields together consistently (e.g. `slot.isBooked` AND `slot.bookedBy` together), atomics alone can't help — that needs a real lock, since atomics only guarantee atomicity of a single variable.

## 🔥 Interview Question
*"AtomicInteger vs synchronized(counter++) — why is Atomic usually faster under contention?"*
→ Atomics use CAS (a CPU instruction, no OS-level thread blocking) — a failed CAS just retries in a tight loop instead of parking the thread, which is cheaper than a lock's block/wake cycle under moderate contention.

## 🔗 Related
[[02 - Locks]]
[[00 - Thread Safety]]

#concurrency
