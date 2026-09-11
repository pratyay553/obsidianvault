---
tags: [concurrency]
---

# Race Conditions

## 🧠 Problem
In load testing, SAloon occasionally double-booked the same slot — but only under real concurrent traffic, never in normal manual testing. The bug was invisible until specifically tested for.

## 🤔 Why does the problem exist
A race condition happens when the correctness of a result depends on the *timing/interleaving* of threads — the code "usually" works because threads usually don't collide at exactly the wrong instant, which is exactly what makes these bugs so dangerous: they hide until production load exposes them.

## 💡 Idea
Identify every "check-then-act" or "read-modify-write" sequence on shared state, and make each one atomic (via locks, atomics, or by re-designing to avoid shared mutable state at all).

## 🏗️ Structure
```
Thread A: read slot.isBooked (false)
                              Thread B: read slot.isBooked (false)  ← same read, both see "free"
Thread A: write slot.isBooked = true
                              Thread B: write slot.isBooked = true  ← double-booked
```

## ☕ Java Implementation — finding and fixing it
```java
// Race condition (looks fine, isn't):
if (!slot.isBooked()) {       // check
    slot.markBooked();         // act — gap between check and act is exploitable
}

// Fixed — check-then-act made atomic:
synchronized (slotLock) {
    if (!slot.isBooked()) slot.markBooked();
}
```

## 🏢 SAloon Example
This is the exact bug traced in [[../04 - UML/01 - Sequence Diagram]] and fixed with [[00 - Thread Safety]]'s `SlotLockManager` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. It only appeared under concurrent load testing, never manual QA — a good reminder to always load-test booking/inventory-style systems specifically.

## 🎯 When should I look for this?
Any time two requests can plausibly hit the same shared resource close together in time — inventory counts, slot bookings, unique-code generation, "first come first served" logic.

## ⚠️ What doesn't fix it?
Adding a delay, retry, or "just check twice" — none of these close the actual timing gap; only real synchronization (locks/atomics/DB transactions) does.

## 🔥 Interview Question
*"How would you even find a race condition like this before it hits production?"*
→ Load/stress test with concurrent requests targeting the *same* resource specifically (not just high overall traffic) — most race conditions need contention on the *same* row/slot/counter to manifest, so random load alone often won't trigger them.

## 🔗 Related
[[00 - Thread Safety]]
[[../04 - UML/01 - Sequence Diagram]]

#concurrency
