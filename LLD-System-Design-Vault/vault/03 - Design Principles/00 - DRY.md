---
tags: [principle]
---

# DRY — Don't Repeat Yourself

## 🧠 Problem
The rule "a booking can't be cancelled within 2 hours of the slot" is checked in `BookingController`, and I just found the same check copy-pasted in `AdminCancelBookingService` too — with a typo, it's `3 hours` there.

## 🤔 Why does the problem exist
Every duplicate is a place the same bug fix or rule change has to be applied. Miss one, and you get silent inconsistency — exactly what happened above.

## 💡 Idea
Every piece of knowledge/business rule should have a single, authoritative representation in the system. Not "don't repeat code" literally — repeat *knowledge* never, but two pieces of code that look similar by coincidence don't need forced unification.

## 🏗️ Structure
```
BookingController  ─┐
AdminCancelService  ─┼──► Booking.canCancel()   ← single source of truth
```

## ☕ Java Implementation
```java
class Booking {
    private static final Duration CANCEL_WINDOW = Duration.ofHours(2);
    boolean canCancel(Instant now) { return Duration.between(now, slotTime).compareTo(CANCEL_WINDOW) > 0; }
}
```
Both callers call `booking.canCancel(now)` — the rule lives in exactly one place.

## 🏢 SAloon Example
[[09 - Real Project LLD/SAloon/00 - Booking]] owns every rule about its own lifecycle — no service outside `Booking` re-implements "is this cancellable."

## 🎯 When should I use it?
For actual **business rules and knowledge** — pricing formulas, validation rules, constants with meaning.

## ⚠️ When NOT to use it?
Don't force two *accidentally* similar code blocks into one shared function if they represent different concepts that just happen to look alike today — duplication is cheaper than the wrong abstraction.

## 🔥 Interview Question
*"Is DRY about code or about knowledge?"*
→ Knowledge. Two identical-looking `if (x > 10)` checks that mean different things in different contexts are not a DRY violation.

## 🔗 Related
[[01 - KISS]]
[[02 - SOLID/00 - SRP]]

#principle
