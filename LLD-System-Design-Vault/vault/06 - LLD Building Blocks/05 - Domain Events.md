---
tags: [building-block]
---

# Domain Events

## 🧠 Problem
When a SAloon booking is cancelled, several downstream things need to happen — refund, notify stylist, free the slot, log analytics — and they don't all belong inside `Booking.cancel()` itself; that would make `Booking` know about refunds and analytics, which isn't its job.

## 🤔 Why does the problem exist
Direct calls from `Booking.cancel()` to `RefundService`, `NotificationService`, etc. would violate [[../02 - SOLID/00 - SRP|SRP]] — `Booking` would need to know about every downstream concern.

## 💡 Idea
When something significant happens, the entity raises a Domain Event (a small immutable record describing "what happened"). Interested handlers subscribe to that event type and react — the entity itself stays ignorant of who's listening (this is [[../05 - Design Patterns/Behavioral/Observer|Observer]] at the architecture level, not just the object level).

## 🏗️ Structure
```
Booking.cancel() → raises → BookingCancelledEvent
                                  ├── RefundHandler
                                  ├── SlotReleaseHandler
                                  └── NotificationHandler
```

## ☕ Java Implementation
```java
record BookingCancelledEvent(UUID bookingId, Instant occurredAt) {}

class Booking {
    private List<Object> pendingEvents = new ArrayList<>();
    void cancel() {
        this.status = CANCELLED;
        pendingEvents.add(new BookingCancelledEvent(id, Instant.now()));
    }
    List<Object> pullEvents() { var e = pendingEvents; pendingEvents = new ArrayList<>(); return e; }
}
// An EventDispatcher (outside Booking) reads pulled events and routes them to handlers.
```

## 🏢 SAloon Example
`BookingCancelledEvent` triggers refund + slot release + notification independently — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. Adding a new reaction (say, a loyalty-points reversal) is a new handler, zero changes to `Booking`.

## 🎯 When should I use it?
An action has multiple, growing, independent downstream reactions that shouldn't be tightly coupled to the entity triggering them.

## ⚠️ When NOT to use it?
A single, fixed downstream reaction doesn't need the machinery of an event system — a direct call is simpler to trace and debug.

## 🔥 Interview Question
*"How is a Domain Event different from the Observer pattern?"*
→ Same underlying idea (decoupled notification), but Domain Events are usually immutable data records describing *what happened in the past* (often used across process/service boundaries, even async queues), while Observer is typically synchronous, in-process object references calling each other directly.

## 🔗 Related
[[../05 - Design Patterns/Behavioral/Observer]]
[[02 - Service]]

#building-block
