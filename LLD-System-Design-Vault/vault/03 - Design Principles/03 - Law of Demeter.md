---
tags: [principle]
---

# Law of Demeter (Principle of Least Knowledge)

## 🧠 Problem
`booking.getCustomer().getWallet().getBalance().subtract(amount)` — this chain lives in `BookingService`, which now silently depends on the *entire internal structure* of `Customer` and `Wallet`.

## 🤔 Why does the problem exist
If `Customer` ever stops holding a `Wallet` directly, every caller with a chain like this breaks. The caller knows way more about `Customer`'s internals than it should.

## 💡 Idea
"Talk only to your immediate friends." An object should only call methods on: itself, objects passed to it, objects it creates, or objects it directly holds — not on the internals of those objects.

## 🏗️ Structure
```
Bad:  BookingService → Customer → Wallet → Balance
Good: BookingService → customer.pay(amount)   ← Customer hides how payment happens
```

## ☕ Java Implementation
```java
// Bad:
booking.getCustomer().getWallet().getBalance().subtract(amount);

// Good:
class Customer {
    void pay(double amount) { wallet.debit(amount); }
}
booking.getCustomer().pay(amount);
```

## 🏢 SAloon Example
`BookingService` calls `customer.pay(amount)` — never reaches through to `Wallet` directly. When SAloon moved balances to a real ledger service, only `Customer.pay()` changed. See [[09 - Real Project LLD/SAloon/01 - Payment]].

## 🎯 When should I use it?
Whenever you see more than one `.` chained through *different* objects' internals.

## ⚠️ When NOT to use it?
Fluent builders (`new Booking.Builder().slot(s).customer(c).build()`) look like chains but call methods on the *same* object repeatedly — that's fine.

## 🔥 Interview Question
*"How does Law of Demeter relate to encapsulation?"*
→ It's encapsulation applied one level further out — not just "hide your own fields," but "don't force callers to know the shape of objects you expose."

## 🔗 Related
[[01 - OOP Fundamentals/00 - Encapsulation]]
[[04 - Composition over Inheritance]]

#principle
