---
tags: [pattern, behavioral]
---

# Strategy

## 🧠 Problem
SAloon prices a booking differently on weekends, festivals, and for first-time customers. `calculatePrice()` was turning into a wall of `if/else`.

## 🤔 Why does the problem exist
Every new pricing rule means editing the same method and risking breaking rules that already worked — this is exactly the [[02 - SOLID/01 - OCP|OCP]] violation OCP warns about.

## 💡 Idea
Encapsulate each algorithm (each pricing rule) in its own class implementing a shared interface. The caller holds a reference to *a* strategy and swaps it at runtime without knowing which concrete one it is.

## 🏗️ Structure
```
PricingStrategy (interface)
   ├── NormalPricing
   ├── WeekendPricing
   └── FestivalPricing
BookingService --uses--> PricingStrategy
```

## ☕ Java Implementation
```java
interface PricingStrategy { double price(Booking b); }
class NormalPricing implements PricingStrategy { public double price(Booking b) { return b.base(); } }
class WeekendPricing implements PricingStrategy { public double price(Booking b) { return b.base() * 1.2; } }

class BookingService {
    private PricingStrategy strategy;
    BookingService(PricingStrategy strategy) { this.strategy = strategy; }
    double checkout(Booking b) { return strategy.price(b); }
}
```

## 🏢 SAloon Example
`PricingStrategy` — see [[09 - Real Project LLD/SAloon/04 - Pricing]]. Also `SplitStrategy`-style logic for staff commission calculation uses the same shape.

## 🎯 When should I use it?
Multiple ways to do the same task, chosen at runtime, and the list is expected to grow — pairs almost always with [[../Creational/Factory|Factory]] to *pick* the strategy.

## ⚠️ When NOT to use it?
Two fixed algorithms that will never grow — a plain `if/else` reads faster than a whole interface hierarchy for something that stable.

## 🔥 Interview Question
*"Strategy vs State — the UML diagram looks identical, what's different?"*
→ Strategy is chosen by the *client* (I pick which strategy to inject). State transitions *itself* based on internal logic (an object moves from one state to another without the client deciding).

## 🔗 Related
[[../../02 - SOLID/01 - OCP]]
[[State]]
[[../Creational/Factory]]

#pattern/behavioral
