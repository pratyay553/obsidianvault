---
tags: [oop]
---

# Encapsulation

## 🧠 Problem
I have data (a bank balance, a booking's seat count) and I want to stop *any random piece of code* from setting it to something invalid.

## 🤔 Why does the problem exist
If every field is `public`, there's nothing stopping `account.balance = -500;` from anywhere in the codebase. The class has no way to defend its own rules. Bugs like this are almost impossible to trace back because the mutation could've happened anywhere.

## 💡 Idea
Hide the data. Force every read/write to go through methods the class controls. The object becomes the only thing that can break its own invariants — and it won't, because it validates.

## 🏗️ Structure
```
Object
 ├── private fields   (hidden)
 └── public methods   (the only door in)
```

## ☕ Java Implementation
```java
class BankAccount {
    private double balance;

    public void withdraw(double amt) {
        if (amt > balance) throw new IllegalStateException("insufficient funds");
        balance -= amt;
    }
    public double getBalance() { return balance; }
}
```
Notice there's no `setBalance()`. If I need one, it means I haven't found the real *behavior* yet.

## 🏢 SAloon Example
`Booking.confirm()` — not `booking.setStatus(CONFIRMED)`. The booking decides for itself whether it's *allowed* to transition to confirmed (slot still free? payment done?). See [[09 - Real Project LLD/SAloon/00 - Booking]].

## 🎯 When should I use it?
Always, by default. Every field starts `private` until I have a reason to expose it.

## ⚠️ When NOT to use it?
Pure data-transfer objects (DTOs) crossing a network boundary — there, plain fields/records are fine because there's no behavior to protect.

## 🔥 Interview Question
*"Why not just make everything public and skip the getters/setters?"*
→ Because a getter/setter pair that just mirrors the field is **not** encapsulation — it's the same bug with extra steps. Real encapsulation means the method *enforces a rule*, not just moves data.

## 🔗 Related
[[01 - Abstraction]]
[[06 - LLD Building Blocks/00 - Entity vs Value Object]]
[[02 - SOLID/00 - SRP]]

#oop
