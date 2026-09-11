---
tags: [building-block]
---

# Factory (as a Building Block, not just a GoF Pattern)

## 🧠 Problem
Constructing a fully valid `Booking` requires computing a price, generating an ID, and validating slot availability — that's too much for a plain constructor call scattered across the codebase.

## 🤔 Why does the problem exist
If every call site does `new Booking(...)` and then separately fills in derived fields (price, ID), it's easy for one call site to forget a step and produce a half-valid object.

## 💡 Idea
Push all the "how do I build a fully valid one of these" logic into one factory-style method/class, separate from the [[../05 - Design Patterns/Creational/Factory|GoF Factory Method pattern]] concept but same spirit — one trusted place that knows how to correctly construct the object.

## 🏗️ Structure
```
BookingFactory.create(customer, slot, pricingStrategy) → fully valid Booking
```

## ☕ Java Implementation
```java
class BookingFactory {
    static Booking create(Customer c, Slot s, PricingStrategy pricing) {
        double price = pricing.price(s);
        return new Booking(UUID.randomUUID(), c, s, price, BookingStatus.PENDING);
    }
}
```

## 🏢 SAloon Example
`BookingFactory` centralizes ID generation + initial pricing so `Booking`'s constructor itself can stay `private`/package-private — see [[../09 - Real Project LLD/SAloon/00 - Booking]].

## 🎯 When should I use it?
Constructing an object requires computed/derived fields, or validation that spans multiple inputs, beyond what a constructor alone should reasonably do.

## ⚠️ When NOT to use it?
A class with a trivial constructor doesn't need a factory wrapping it — that's needless indirection.

## 🔥 Interview Question
*"Where's the line between this and the Builder pattern?"*
→ Factory: "which/how do I construct this one thing, possibly with derived logic." [[../05 - Design Patterns/Creational/Builder|Builder]]: "let me assemble this thing step by step with lots of optional fields." Often used together — a Builder's `.build()` can internally call a Factory.

## 🔗 Related
[[../05 - Design Patterns/Creational/Factory]]
[[../05 - Design Patterns/Creational/Builder]]

#building-block
