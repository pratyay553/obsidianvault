---
tags: [building-block]
---

# Entity vs Value Object

## 🧠 Problem
Is a `Booking` the same as another `Booking` if all its fields match? Is a `Money` amount of ₹500 the same as another ₹500?

## 🤔 Why does the problem exist
Treating everything as "equal if fields match" is wrong for things with identity (two bookings for the same slot/customer/time are still *different* bookings if made twice by mistake — that's actually a bug to catch, not a duplicate to merge). But treating everything as "equal only if same reference" is wrong for things like money, where two ₹500s are genuinely interchangeable.

## 💡 Idea
- **Entity**: has an identity (an ID) that persists across changes. Two entities are equal only if their IDs match, even if every other field is identical.
- **Value Object**: has no identity — equality is based purely on its field values. Immutable by convention.

## 🏗️ Structure
```
Booking (Entity)              Money (Value Object)
  - id: UUID  ← identity          - amount: double
  - status                        - currency: String
  equals() → compares id          equals() → compares amount + currency
```

## ☕ Java Implementation
```java
class Booking { // Entity
    private final UUID id;
    private BookingStatus status;
    @Override public boolean equals(Object o) {
        return o instanceof Booking b && this.id.equals(b.id);
    }
}

final class Money { // Value Object — immutable, no identity
    private final double amount;
    private final String currency;
    Money(double amount, String currency) { this.amount = amount; this.currency = currency; }
    @Override public boolean equals(Object o) {
        return o instanceof Money m && amount == m.amount && currency.equals(m.currency);
    }
}
```

## 🏢 SAloon Example
`Booking` is an Entity (has a persistent ID, tracked across its lifecycle). `Money` (used for pricing) and `TimeSlot` (a start/end time pair) are Value Objects — see [[../09 - Real Project LLD/SAloon/04 - Pricing]].

## 🎯 When should I use Value Objects?
Anything defined entirely by its attributes with no need to track "which specific instance" — money, coordinates, date ranges, addresses.

## ⚠️ When NOT to use it?
Don't make something a Value Object if the system genuinely needs to track it individually over time (e.g. two otherwise-identical bookings are still separately cancellable).

## 🔥 Interview Question
*"Why make Value Objects immutable?"*
→ If `Money` were mutable, two references pointing at "the same ₹500" could diverge unexpectedly when one caller mutates it — since value objects are meant to be freely shared/copied, immutability is what makes that safe.

## 🔗 Related
[[../01 - OOP Fundamentals/00 - Encapsulation]]
[[01 - Repository]]

#building-block
