---
tags: [saloon, real-project]
---

# SAloon — Booking Module

## 🧠 Problem
A customer needs to book a specific service with a specific stylist at a specific time slot — and the whole system falls apart if two customers can book the same stylist's same time slot.

## 🤔 Why does the problem exist
This is the load-bearing module of the whole app — everything else (payment, pricing, notification) exists to support a correct booking. It's also the one place every concurrency bug in this vault actually showed up first, under real load testing.

## 💡 Idea
`Booking` is an [[../../06 - LLD Building Blocks/00 - Entity vs Value Object|Entity]] that owns its own lifecycle via [[../../05 - Design Patterns/Behavioral/State|State]]. `BookingFacade` orchestrates the full checkout across services. `SlotLockManager` makes slot allocation atomic.

## 🏗️ Structure
```
Booking (Entity)
  - id, customer, slot, status: BookingState
  + confirm(), cancel()

BookingState (interface) → PendingState, ConfirmedState, CompletedState, CancelledState

BookingFacade.book(request)
  ├── AvailabilityChecker.isFree(slot)
  ├── SlotLockManager.tryBook(slot)      ← synchronized, per-slot lock
  ├── PricingService.calculate(booking)   ← Strategy + Decorator stack
  ├── PaymentGateway.pay(customer, price) ← Adapter over Razorpay/Stripe
  ├── BookingRepository.save(booking)
  └── observers.notifyAll() (confirmation) ← Observer
```

## ☕ Java Implementation (the critical section)
```java
class SlotLockManager {
    private final Map<UUID, Object> locks = new ConcurrentHashMap<>();
    boolean tryBook(Slot slot) {
        Object lock = locks.computeIfAbsent(slot.getId(), k -> new Object());
        synchronized (lock) {
            if (slot.isBooked()) return false;
            slot.markBooked();
            return true;
        }
    }
}

class BookingFacade {
    Booking book(BookingRequest req) {
        if (!availability.isFree(req.slot())) throw new SlotUnavailableException();
        if (!slotLockManager.tryBook(req.slot())) throw new SlotUnavailableException(); // atomic re-check
        double price = pricing.calculate(req);
        payment.pay(req.customer(), price); // AFTER lock, BEFORE confirm — see sequence diagram
        Booking booking = bookingFactory.create(req, price);
        repo.save(booking);
        booking.confirm();
        return booking;
    }
}
```

## The bug I actually hit
Originally charged the customer *before* locking the slot. Under load, two customers could both pass the availability check, both get charged, and only one could actually get the slot — meaning a refund flow I hadn't built yet. Drawing the [[../../04 - UML/01 - Sequence Diagram|sequence diagram]] made the fix (lock before charge, always) obvious.

## Every pattern this module uses
- [[../../05 - Design Patterns/Behavioral/State]] — booking lifecycle
- [[../../05 - Design Patterns/Structural/Facade]] — `BookingFacade`
- [[../../05 - Design Patterns/Creational/Builder]] — assembling a `Booking` with optional fields
- [[../../05 - Design Patterns/Creational/Singleton]] — `AppointmentScheduler`
- [[../../05 - Design Patterns/Behavioral/Observer]] — confirmation notifications
- [[../../05 - Design Patterns/Behavioral/Command]] — admin cancel/refund actions, audit trail
- [[../../06 - LLD Building Blocks/05 - Domain Events]] — `BookingCancelledEvent` fan-out
- [[../../07 - Concurrency/00 - Thread Safety]] — `SlotLockManager`

## 🔥 Interview Question
*"Walk me through what happens, end to end, when a booking request comes in."*
→ This note's structure diagram, plus the sequence-diagram lesson above about lock-before-charge, is exactly the answer.

## 🔗 Related
[[01 - Payment]]
[[02 - Notification]]
[[04 - Pricing]]

#saloon
