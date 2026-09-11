---
tags: [principle]
---

# Composition over Inheritance (Principle)

*(See [[01 - OOP Fundamentals/04 - Composition vs Inheritance]] for the full OOP-level treatment — this note is the short rule I actually apply while designing.)*

## 🧠 Problem
Do I model `WeekendStylist extends Stylist` for a stylist who charges differently on weekends?

## 🤔 Why does the problem exist
That would mean a new subclass for every combination of role × pricing-rule × availability-rule — a combinatorial explosion.

## 💡 Idea
Give `Stylist` a `PricingStrategy` field instead of subclassing per pricing variant. Compose behavior in, don't inherit it.

## 🏗️ Structure
```
Stylist
  └── has-a → PricingStrategy (swap freely, no new class hierarchy)
```

## ☕ Java Implementation
```java
class Stylist {
    private PricingStrategy pricing; // composed
    Stylist(PricingStrategy pricing) { this.pricing = pricing; }
}
```

## 🏢 SAloon Example
Every stylist in SAloon is one class — variation comes entirely from injected `PricingStrategy` + `AvailabilityRule` objects, not subclasses. See [[09 - Real Project LLD/SAloon/04 - Pricing]].

## 🎯 When should I use it?
The default. Reach for inheritance only when you've checked it against [[02 - SOLID/02 - LSP]] and it still holds.

## ⚠️ When NOT to use it?
Never really "shouldn't" — worst case, composition adds one extra field vs inheritance's zero, so the cost of defaulting to it is low.

## 🔥 Interview Question
*"Give a concrete sign in a design that inheritance was the wrong call."*
→ A subclass that overrides a method to throw `UnsupportedOperationException`, or an override that does nothing just to "cancel out" inherited behavior.

## 🔗 Related
[[01 - OOP Fundamentals/04 - Composition vs Inheritance]]
[[05 - Design Patterns/Behavioral/Strategy]]

#principle
