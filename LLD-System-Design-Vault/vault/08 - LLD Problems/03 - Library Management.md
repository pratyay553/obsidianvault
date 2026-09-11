---
tags: [lld-problem]
---

# Library Management System

## 🧠 Problem
Design a system to catalog books (with multiple physical copies per title), let members borrow/return/reserve them, and calculate fines for overdue returns.

## 🤔 Why does the problem exist
The tricky part isn't borrowing itself — it's correctly separating a *title* (metadata, shared) from a *physical copy* (has its own status: available/loaned/reserved). Conflating these means "how many copies are left" becomes error-prone to track.

## 💡 Idea
`Book` (title/metadata) is distinct from `BookItem` (one physical copy with its own status). The borrow/return workflow is shared across media types via [[../05 - Design Patterns/Behavioral/Template Method|Template Method]], with fine calculation pluggable per type.

## 🏗️ Structure
```
Book (title, author)  1 ── * BookItem (barcode, status)
Member — has → List<Loan>
FineStrategy (interface) → BookFineStrategy, DvdFineStrategy
```

## ☕ Java Implementation
```java
class BookItem {
    enum Status { AVAILABLE, LOANED, RESERVED }
    private Status status = Status.AVAILABLE;
    synchronized boolean checkout() {
        if (status != Status.AVAILABLE) return false;
        status = Status.LOANED;
        return true;
    }
}
```

## 🏢 SAloon Tie-in
`Book` vs `BookItem` mirrors a `ServiceType` (e.g. "Haircut," metadata) vs an actual bookable `Slot` for a specific stylist — see [[../09 - Real Project LLD/SAloon/00 - Booking]].

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/Strategy]] — fine calculation per item type
- [[../05 - Design Patterns/Behavioral/Observer]] — notify waitlisted member when a reserved copy returns

## 🔥 Interview Question
*"How do you prevent two members checking out the last copy simultaneously?"*
→ Same shape as every allocation problem in this vault — make `checkout()` atomic ([[../07 - Concurrency/00 - Thread Safety]]), don't just check-then-act unprotected.

## 🔗 Related
[[../07 - Concurrency/00 - Thread Safety]]
[[../06 - LLD Building Blocks/00 - Entity vs Value Object]]

#lld-problem
