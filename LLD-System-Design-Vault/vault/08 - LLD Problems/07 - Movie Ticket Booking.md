---
tags: [lld-problem]
---

# Movie Ticket Booking System

## 🧠 Problem
Design a BookMyShow-style system: browse shows, select seats, hold them temporarily during checkout, pay, and confirm — without letting two users book the same seat.

## 🤔 Why does the problem exist
This is functionally the *exact same problem* as SAloon's slot booking, just with seats instead of stylist time slots — same double-booking race condition, same need for a temporary "lock" window during checkout before payment completes.

## 💡 Idea
Seats have their own `SeatState` (Available/Locked/Booked). A lock is temporary (expires if payment isn't completed within N minutes) — model this with a scheduled release, not just a permanent flag flip.

## 🏗️ Structure
```
Seat — state: SeatState (Available/Locked/Booked)
BookingFacade.book(seats, show) → lock → charge → confirm
SeatLockManager — holds locks with expiry, releases on timeout
```

## ☕ Java Implementation (temporary lock)
```java
class SeatLockManager {
    private Map<UUID, ScheduledFuture<?>> expiryTasks = new ConcurrentHashMap<>();
    boolean lock(Seat seat, Duration ttl) {
        synchronized (seat) {
            if (seat.getState() != SeatState.AVAILABLE) return false;
            seat.setState(SeatState.LOCKED);
        }
        var task = scheduler.schedule(() -> release(seat), ttl.toMillis(), MILLISECONDS);
        expiryTasks.put(seat.getId(), task);
        return true;
    }
}
```

## 🏢 SAloon Tie-in
This *is* SAloon's booking problem — see [[../09 - Real Project LLD/SAloon/00 - Booking]] for the near-identical `SlotLockManager` with the same temporary-hold requirement during checkout.

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/State]] — seat states
- [[../05 - Design Patterns/Structural/Facade]] — `BookingFacade`
- [[../05 - Design Patterns/Creational/Builder]] — building a multi-seat `Booking`

## 🔥 Interview Question
*"What happens if payment succeeds right as the lock expires?"*
→ Classic race condition — the payment-confirmation step must atomically check "is this lock still mine and not expired" before finalizing, and the expiry task must be cancelled the moment payment succeeds, not left to fire independently.

## 🔗 Related
[[../07 - Concurrency/05 - Race Conditions]]
[[../09 - Real Project LLD/SAloon/00 - Booking]]

#lld-problem
