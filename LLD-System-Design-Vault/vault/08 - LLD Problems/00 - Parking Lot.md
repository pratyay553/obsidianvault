---
tags: [lld-problem]
---

# Parking Lot

## 🧠 Problem
Design a multi-floor parking lot that assigns spots to different vehicle sizes, tracks occupancy, and calculates a fee on exit — without letting two vehicles get assigned the same spot.

## 🤔 Why does the problem exist
This is really a resource-allocation problem under concurrency, dressed up as a domain — the same shape as [[03 - Elevator]] and SAloon's slot booking. Interviewers use it to see if you separate *allocation logic*, *pricing logic*, and *entity modeling* cleanly instead of writing one giant `ParkingLot` god-class.

## 💡 Idea
Model `ParkingSpot` as owning its own occupancy state, use a `Singleton` manager for the lot-wide view, and make pricing pluggable.

## 🏗️ Structure
```
ParkingLot (Singleton)     — coordinates floors
  └── Floor
        └── ParkingSpot (- isOccupied, - type)
Vehicle (abstract) → Car, Motorcycle, Bus
PricingStrategy (interface) → HourlyPricing, FlatRatePricing
```

## ☕ Java Implementation (core allocation)
```java
class ParkingSpot {
    private boolean occupied;
    private final SpotType type;
    synchronized boolean assign(Vehicle v) {
        if (occupied || !type.fits(v)) return false;
        occupied = true;
        return true;
    }
}
```

## 🏢 SAloon Tie-in
Structurally identical to SAloon's slot-booking problem — `ParkingSpot.assign()` here is `Slot.tryBook()` there, both needing the same [[../07 - Concurrency/00 - Thread Safety|thread-safety]] treatment. See [[../09 - Real Project LLD/SAloon/00 - Booking]].

## Patterns Applied
- [[../05 - Design Patterns/Creational/Singleton]] — one `ParkingLot` instance
- [[../05 - Design Patterns/Behavioral/Strategy]] — swappable `PricingStrategy`
- [[../05 - Design Patterns/Structural/Facade]] — `ParkingLotFacade` for entry/exit flow

## 🔥 Interview Question
*"How do you find the nearest available spot efficiently?"*
→ Maintain a free-spot index (e.g. a `TreeSet` or per-type queue) per floor instead of scanning every spot — O(log n) instead of O(n) per assignment.

## 🔗 Related
[[03 - Elevator]]
[[../07 - Concurrency/00 - Thread Safety]]

#lld-problem
