---
tags: [saloon, real-project]
---

# SAloon — Review Module

## 🧠 Problem
Customers leave a rating + text review after a completed booking — but a review should only be postable once per booking, only after the booking is actually `COMPLETED`, and the salon's average rating needs to update without recomputing from every review each time.

## 🤔 Why does the problem exist
If review-eligibility logic ("is this booking postable-to") is checked inline in the controller, it's easy for a different code path (say, an admin tool) to skip the check and create a duplicate or premature review.

## 💡 Idea
`Booking` itself exposes `canReview()` (an [[../../01 - OOP Fundamentals/00 - Encapsulation|encapsulated]] rule, same idea as `canCancel()`). `Review` is a [[../../06 - LLD Building Blocks/00 - Entity vs Value Object|Value-Object-like]] immutable record once created. Salon rating is maintained incrementally, not recomputed from scratch.

## 🏗️ Structure
```
Booking
  + canReview(): boolean   // status == COMPLETED && !hasReview

Review (immutable) — bookingId, rating, text, createdAt

Salon
  - reviewCount, ratingSum   ← incremental, O(1) average update
  + addReview(rating) { reviewCount++; ratingSum += rating; }
  + averageRating() { return ratingSum / reviewCount; }
```

## ☕ Java Implementation
```java
class Booking {
    boolean canReview() { return status == BookingStatus.COMPLETED && !hasReview; }
}

class Salon {
    private int reviewCount = 0;
    private double ratingSum = 0;
    void addReview(int rating) { reviewCount++; ratingSum += rating; } // O(1), no rescan
    double averageRating() { return reviewCount == 0 ? 0 : ratingSum / reviewCount; }
}
```

## Why incremental average, not recompute-from-all-reviews
A popular salon can have tens of thousands of reviews — recomputing the average on every new review is an avoidable O(n) cost for something that's naturally O(1) if tracked incrementally. Small decision, but it's the kind of thing interviewers listen for.

## Every pattern/principle this module uses
- [[../../01 - OOP Fundamentals/00 - Encapsulation]] — `Booking.canReview()`
- [[../../06 - LLD Building Blocks/00 - Entity vs Value Object]] — `Review` as an immutable value
- [[../../03 - Design Principles/00 - DRY]] — single source of truth for review eligibility

## 🔥 Interview Question
*"How would you prevent a race where two review submissions for the same booking both pass the `canReview()` check?"*
→ Same family of bug as every other allocation problem in this vault — the check-then-insert needs to be atomic (a unique DB constraint on `bookingId` in the `Review` table is the simplest real-world fix, backed up by an application-level lock for the earlier fail-fast check).

## 🔗 Related
[[00 - Booking]]
[[../../07 - Concurrency/05 - Race Conditions]]

#saloon
