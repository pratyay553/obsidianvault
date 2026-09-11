---
tags: [solid]
---

# Liskov Substitution Principle

## 🧠 Problem
I have `OnlinePayment` and `CashPayment` both extending `Payment`. `CashPayment.refund()` throws `UnsupportedOperationException` because cash can't be auto-refunded. Code that calls `payment.refund()` generically now crashes for one subtype.

## 🤔 Why does the problem exist
If a subclass can't honestly fulfill the parent's contract, any code written against the parent type becomes unsafe to call on the subclass — you have to know the concrete type to use it correctly, which defeats the entire point of polymorphism.

## 💡 Idea
Subtypes must be substitutable for their base type — anywhere a `Payment` is expected, *any* subclass should work without the caller needing special-case knowledge.

## 🏗️ Structure
```
Bad:                          Good:
Payment                       Payment
 ├─ OnlinePayment                ├─ RefundablePayment
 └─ CashPayment (refund          │    └─ OnlinePayment
     throws!)                    └─ CashPayment (no refund method at all)
```

## ☕ Java Implementation
```java
interface Payment { void pay(double amt); }
interface Refundable { void refund(double amt); }

class OnlinePayment implements Payment, Refundable { ... }
class CashPayment implements Payment { ... } // simply doesn't claim to be refundable
```
Callers that need refund logic ask for `Refundable` specifically — `CashPayment` was never forced to lie about supporting it.

## 🏢 SAloon Example
`CashPayment` vs `OnlinePayment` in [[09 - Real Project LLD/SAloon/01 - Payment]] is the literal example above — I hit this exact bug in the first draft.

## 🎯 When should I use it?
Every time I design an inheritance/interface hierarchy — ask "can every subtype honestly do everything the contract promises, with no surprises?"

## ⚠️ When NOT to use it?
N/A — LSP isn't optional the way some principles are situational. If it's violated, the hierarchy is wrong, full stop.

## 🔥 Interview Question
*"Square extends Rectangle — why is this the textbook LSP violation?"*
→ `Rectangle.setWidth()` is expected to leave height unchanged. `Square` must change both to stay a square, breaking that expectation for any code written against `Rectangle`.

## 🔗 Related
[[01 - OOP Fundamentals/02 - Inheritance]]
[[03 - ISP]]

#solid
