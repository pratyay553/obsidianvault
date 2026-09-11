---
tags: [saloon, real-project]
---

# SAloon — Pricing Module

## 🧠 Problem
A booking's final price depends on a base rate, plus any combination of: weekend/festival multiplier, tax, peak-hour surcharge, and loyalty/first-time discounts — and new pricing rules get added every few months.

## 🤔 Why does the problem exist
The first `calculatePrice()` was one method with a growing wall of `if/else` for every rule combination — a direct [[../../02 - SOLID/01 - OCP|OCP]] violation, and every new rule risked breaking an existing one.

## 💡 Idea
The *base* rate selection (normal/weekend/festival) is a [[../../05 - Design Patterns/Behavioral/Strategy|Strategy]] — exactly one applies. The *additive* modifiers (tax, surcharge, discount) are [[../../05 - Design Patterns/Structural/Decorator|Decorators]] — any combination can stack.

## 🏗️ Structure
```
PricingStrategy (interface) → NormalPricing, WeekendPricing, FestivalPricing
       ↓ produces a BasePrice, then wrapped by:
PriceComponent (interface) → BasePrice
   ↑ wrapped by TaxDecorator
   ↑ wrapped by PeakHourSurcharge
   ↑ wrapped by LoyaltyDiscount
```

## ☕ Java Implementation
```java
interface PricingStrategy { double basePrice(Booking b); }
class WeekendPricing implements PricingStrategy {
    public double basePrice(Booking b) { return b.getService().getRate() * 1.2; }
}

interface PriceComponent { double getPrice(); }
class BasePrice implements PriceComponent {
    private double amount;
    BasePrice(double amount) { this.amount = amount; }
    public double getPrice() { return amount; }
}
abstract class PriceDecorator implements PriceComponent {
    protected PriceComponent wrapped;
    PriceDecorator(PriceComponent w) { wrapped = w; }
}
class TaxDecorator extends PriceDecorator {
    TaxDecorator(PriceComponent w) { super(w); }
    public double getPrice() { return wrapped.getPrice() * 1.18; }
}

// Wiring it together:
double base = pricingStrategy.basePrice(booking);
PriceComponent finalPrice = new TaxDecorator(new PeakHourSurcharge(new BasePrice(base)));
```

## Why two patterns instead of one
I initially tried to cram both concerns into one big `Strategy` per exact combination (`WeekendWithTaxAndSurchargeStrategy`...) — combinatorial explosion. Splitting "which base rate" (exactly one, mutually exclusive → Strategy) from "which add-ons" (any combination, stackable → Decorator) fixed it, following [[../../01 - OOP Fundamentals/04 - Composition vs Inheritance|composition over inheritance]] all the way through.

## Every pattern this module uses
- [[../../05 - Design Patterns/Behavioral/Strategy]] — base rate selection
- [[../../05 - Design Patterns/Structural/Decorator]] — stackable modifiers
- [[../../02 - SOLID/01 - OCP]] — the whole reason this design exists

## 🔥 Interview Question
*"How did you decide what should be a Strategy vs a Decorator here?"*
→ Mutually exclusive, "pick exactly one" variation → Strategy. Independently combinable, "stack zero or more" variation → Decorator. That litmus test generalizes past pricing.

## 🔗 Related
[[00 - Booking]]
[[../../05 - Design Patterns/Behavioral/Strategy]]
[[../../05 - Design Patterns/Structural/Decorator]]

#saloon
