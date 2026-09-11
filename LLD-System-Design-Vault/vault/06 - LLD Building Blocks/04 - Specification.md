---
tags: [building-block]
---

# Specification

## 🧠 Problem
SAloon needs to filter bookings by combinations like "confirmed AND in the next 7 days AND for a specific stylist" — and the exact combination changes per screen/report. Hardcoding one method per combination (`findConfirmedUpcomingForStylist()`) doesn't scale.

## 🤔 Why does the problem exist
Business rules for "what counts as matching" often need to combine (AND/OR/NOT) in ways that can't all be anticipated as named repository methods ahead of time.

## 💡 Idea
Encapsulate a business rule ("is satisfied by") as its own object, and let specifications compose via `and()`/`or()`/`not()` — build complex queries out of simple, reusable, named rules.

## 🏗️ Structure
```
Specification<Booking> (interface)
   ├── ConfirmedSpec
   ├── UpcomingWithinDaysSpec
   └── ForStylistSpec
combined via .and(...)
```

## ☕ Java Implementation
```java
interface Specification<T> { boolean isSatisfiedBy(T t); default Specification<T> and(Specification<T> o) { return t -> this.isSatisfiedBy(t) && o.isSatisfiedBy(t); } }

class ConfirmedSpec implements Specification<Booking> {
    public boolean isSatisfiedBy(Booking b) { return b.getStatus() == CONFIRMED; }
}
class UpcomingWithinDaysSpec implements Specification<Booking> {
    private int days;
    public boolean isSatisfiedBy(Booking b) { return b.getSlot().isWithinDays(days); }
}

Specification<Booking> spec = new ConfirmedSpec().and(new UpcomingWithinDaysSpec(7));
bookings.stream().filter(spec::isSatisfiedBy).toList();
```

## 🏢 SAloon Example
The admin dashboard's booking filters compose specs like this instead of exposing 15 different repository methods — see [[../09 - Real Project LLD/SAloon/00 - Booking]].

## 🎯 When should I use it?
Filtering/querying logic with many combinable business rules that change often, especially when exposed to end users as filters.

## ⚠️ When NOT to use it?
For 1-2 fixed, rarely-changing query shapes, a plain repository method (`findConfirmedBookings()`) is simpler and more discoverable.

## 🔥 Interview Question
*"How is Specification different from just writing a lambda predicate inline?"*
→ Specifications are *named, reusable, and composable* business rules that can be unit tested in isolation and referenced by name elsewhere — an inline lambda can't be reused or given a meaningful name at the call site.

## 🔗 Related
[[01 - Repository]]
[[05 - Domain Events]]

#building-block
