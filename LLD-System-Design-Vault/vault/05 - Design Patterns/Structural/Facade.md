---
tags: [pattern, structural]
---

# Facade

## 🧠 Problem
Completing a SAloon booking requires: checking availability, locking the slot, applying pricing, charging payment, saving the booking, and sending a notification — six different objects. `BookingController` shouldn't orchestrate all six itself.

## 🤔 Why does the problem exist
Without a coordinating layer, the controller ends up calling six services directly, in the right order, with the right error handling at each step.

## 💡 Idea
Provide one simple entry point that internally coordinates the whole subsystem. The caller sees one method call.

## 🏗️ Structure
```
Client → BookingFacade.book(request)
              ├── AvailabilityChecker
              ├── SlotLockManager
              ├── PricingService
              ├── PaymentGateway
              ├── BookingRepository
              └── NotificationService
```

## ☕ Java Implementation
```java
class BookingFacade {
    private AvailabilityChecker availability;
    private PricingService pricing;
    private PaymentGateway payment;
    private BookingRepository repo;
    private NotificationService notifier;

    Booking book(BookingRequest req) {
        if (!availability.isFree(req.slot())) throw new SlotUnavailableException();
        double price = pricing.calculate(req);
        payment.pay(req.customer(), price);
        Booking booking = new Booking(req, price);
        repo.save(booking);
        notifier.notifyConfirmed(booking);
        return booking;
    }
}
```

## 🏢 SAloon Example
`BookingFacade` is the single entry point the API controller calls — see [[09 - Real Project LLD/SAloon/00 - Booking]]. Every subsystem behind it still follows [[02 - SOLID/00 - SRP|SRP]] individually.

## 🎯 When should I use it?
Any workflow that coordinates 3+ services in a specific order.

## ⚠️ When NOT to use it?
Don't build a facade for a subsystem with only one caller and one method.

## 🔥 Interview Question
*"Doesn't Facade violate Single Responsibility — it 'does a lot'?"*
→ No — its one responsibility is *orchestration/sequencing*, not the actual logic of each step, which still lives in its own class.

## 🔗 Related
[[02 - SOLID/00 - SRP]]
[[Adapter]]

#pattern/structural
