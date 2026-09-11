---
tags: [solid]
---

# Dependency Inversion Principle

## 🧠 Problem
`BookingService` does `new MySQLBookingRepository()` right inside its constructor. Now I can't unit test `BookingService` without spinning up a real MySQL instance, and switching to Postgres means editing `BookingService`.

## 🤔 Why does the problem exist
When a high-level module (business logic) directly instantiates a low-level module (a specific DB driver), the *business logic* ends up depending on the *detail* — the thing most likely to change. That's backwards.

## 💡 Idea
Both high-level and low-level modules should depend on an **abstraction**. The high-level module defines the interface it needs; the low-level module implements it. Concrete instances get **injected**, not constructed internally.

## 🏗️ Structure
```
BookingService (high-level)
      ↓ depends on
BookingRepository (interface)
      ↑ implements
MySQLBookingRepository (low-level, swappable)
```

## ☕ Java Implementation
```java
interface BookingRepository { void save(Booking b); }
class MySQLBookingRepository implements BookingRepository { public void save(Booking b) {} }

class BookingService {
    private final BookingRepository repo;
    BookingService(BookingRepository repo) { this.repo = repo; } // injected, not `new`-ed
}
```

## 🏢 SAloon Example
Every service in [[09 - Real Project LLD/SAloon/00 - Booking]] takes its repository and gateway dependencies through the constructor — this is what let me swap `RazorpayGateway` for a `FakePaymentGateway` in tests with zero changes to `PaymentService`.

## 🎯 When should I use it?
Any dependency on something external and swappable: databases, payment gateways, notification providers, third-party APIs.

## ⚠️ When NOT to use it?
Don't inject something that will genuinely never have a second implementation and isn't a testing concern.

## 🔥 Interview Question
*"What's the difference between DIP and just 'using an interface'?"*
→ DIP is specifically about the *direction* of the dependency — high-level code shouldn't know about low-level details at all, even through composition, unless it's via an abstraction it owns/defines.

## 🔗 Related
[[01 - OOP Fundamentals/01 - Abstraction]]
[[05 - Design Patterns/Structural/Adapter]]
[[00 - SRP]]

#solid
