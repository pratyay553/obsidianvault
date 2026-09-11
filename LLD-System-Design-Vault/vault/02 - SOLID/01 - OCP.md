---
tags: [solid]
---

# Open/Closed Principle

## 🧠 Problem
Every time SAloon adds a new pricing rule (festival pricing, loyalty discount), I'm editing the same `calculatePrice()` method and adding another `else if`.

## 🤔 Why does the problem exist
A method with growing `if/else`/`switch` branches on type becomes a shared bottleneck — every new feature means editing tested, already-working code, risking regressions in the branches you didn't mean to touch.

## 💡 Idea
Classes should be **open for extension, closed for modification**. Add new behavior by adding a new class, not by editing an existing one.

## 🏗️ Structure
```
PricingStrategy (interface)
   ├── NormalPricing
   ├── WeekendPricing
   └── FestivalPricing   ← adding this needs zero edits elsewhere
```

## ☕ Java Implementation
```java
interface PricingStrategy { double price(Booking b); }
class NormalPricing implements PricingStrategy { public double price(Booking b) { return b.base(); } }
class FestivalPricing implements PricingStrategy { public double price(Booking b) { return b.base() * 1.5; } }
```
`BookingService` takes a `PricingStrategy` — adding `LoyaltyPricing` tomorrow touches zero existing files.

## 🏢 SAloon Example
See [[09 - Real Project LLD/SAloon/04 - Pricing]] — this exact interface is how I added "first-time customer discount" without touching `NormalPricing` or `WeekendPricing`.

## 🎯 When should I use it?
Anywhere I can already see the list of "types/variants" is going to grow — pricing rules, notification channels, payment methods.

## ⚠️ When NOT to use it?
Don't pre-build an extension point for something with no evidence it'll ever have a second variant — that's speculative generality, see [[03 - Design Principles/02 - YAGNI]].

## 🔥 Interview Question
*"How does OCP actually get implemented in code?"*
→ Through abstraction + polymorphism (interfaces) — it's not a separate mechanism, it's [[01 - OOP Fundamentals/01 - Abstraction]] applied with foresight about what will vary.

## 🔗 Related
[[00 - SRP]]
[[05 - Design Patterns/Behavioral/Strategy]]
[[05 - Design Patterns/Creational/Factory]]

#solid
