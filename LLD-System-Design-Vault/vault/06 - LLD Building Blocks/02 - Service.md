---
tags: [building-block]
---

# Service

## 🧠 Problem
"Book a slot" involves logic that doesn't naturally belong to any single entity — it needs a `Customer`, a `Slot`, a `PricingStrategy`, and a `PaymentGateway` all working together. Where does that orchestration logic live?

## 🤔 Why does the problem exist
Forcing this into one entity (say, `Booking.book(customer, slot, pricing, payment)`) bloats the entity with dependencies it shouldn't own, and entities are usually meant to stay lightweight and persistence-friendly.

## 💡 Idea
A Service is a stateless class that coordinates operations across multiple entities/repositories to fulfill one use case. It holds *behavior*, not persistent state of its own.

## 🏗️ Structure
```
BookingService (stateless)
   ├── uses → BookingRepository
   ├── uses → PricingStrategy
   └── uses → PaymentGateway
```

## ☕ Java Implementation
```java
class BookingService {
    private final BookingRepository repo;
    private final PricingStrategy pricing;
    private final PaymentGateway payment;

    Booking book(Customer c, Slot s) {
        double price = pricing.price(s);
        payment.pay(c, price);
        Booking b = new Booking(c, s, price);
        repo.save(b);
        return b;
    }
}
```

## 🏢 SAloon Example
`BookingService` — see [[../09 - Real Project LLD/SAloon/00 - Booking]]. Note it's stateless: every dependency comes in via constructor, nothing is stored as mutable service-level state (see [[../02 - SOLID/04 - DIP]]).

## 🎯 When should I use it?
Any use case/workflow that spans multiple entities or repositories.

## ⚠️ When NOT to use it?
Don't create a Service for logic that genuinely belongs to a single entity — that's the "anemic domain model" anti-pattern, see [[../01 - OOP Fundamentals/00 - Encapsulation]].

## 🔥 Interview Question
*"How do you decide if logic belongs in an entity or a service?"*
→ If it only needs data the entity already owns, it's the entity's job (`booking.canCancel()`). If it needs to coordinate multiple entities/repositories/external systems, it's a service's job.

## 🔗 Related
[[00 - Entity vs Value Object]]
[[01 - Repository]]
[[../02 - SOLID/00 - SRP]]

#building-block
