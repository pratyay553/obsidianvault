---
tags: [solid]
---

# Single Responsibility Principle

## 🧠 Problem
`Booking` class calculates the price, saves itself to the DB, *and* sends a confirmation email. One small change to the email template means recompiling and re-testing the pricing logic too.

## 🤔 Why does the problem exist
When unrelated responsibilities live in one class, a change driven by *one* stakeholder (marketing wants a new email template) risks breaking something a *different* stakeholder depends on (finance's pricing logic). The class has more than one reason to change.

## 💡 Idea
A class should have exactly **one reason to change**. If I can't describe a class's job without saying "and", it's two classes.

## 🏗️ Structure
```
Before:                     After:
Booking                     Booking (data + rules)
 ├─ calculatePrice()        BookingRepository (persistence)
 ├─ saveToDb()               NotificationService (email/sms)
 └─ sendEmail()
```

## ☕ Java Implementation
```java
class Booking { /* holds state, enforces its own invariants */ }
class BookingRepository { void save(Booking b) { /* DB */ } }
class NotificationService { void notifyConfirmed(Booking b) { /* email/sms */ } }
```

## 🏢 SAloon Example
[[09 - Real Project LLD/SAloon/00 - Booking]] only knows how to validate and transition its own state. Saving it is `BookingRepository`'s job, notifying the customer is `NotificationService`'s job — see [[09 - Real Project LLD/SAloon/02 - Notification]].

## 🎯 When should I use it?
Constantly — it's the default lens I check every class through: "what's the one sentence description of this class's job?"

## ⚠️ When NOT to use it?
Splitting too aggressively creates a maze of tiny classes for a genuinely simple, stable feature. If a class truly has one reason to change even though it does "two things," don't force a split.

## 🔥 Interview Question
*"Isn't SRP just 'one class = one method'?"*
→ No — a class can have many methods as long as they all serve the *same responsibility*. SRP is about reasons-to-change, not method count.

## 🔗 Related
[[01 - OOP Fundamentals/00 - Encapsulation]]
[[01 - OCP]]
[[06 - LLD Building Blocks/02 - Service]]

#solid
