---
tags: [interview-prep]
---

# Interview Checklist (Day-Before Pass)

## 🧠 Problem
The night before an LLD interview, I don't want to re-read the whole vault — I want a fast checklist that confirms I can still explain everything cold.

## 🤔 Why a checklist instead of re-reading
Re-reading creates false confidence (recognition ≠ recall). Forcing myself to explain each item out loud from just the prompt below actually tests recall the way an interview will.

## ✅ Can I explain each of these out loud, no notes?
- [ ] The four OOP pillars, with a "why does the problem exist" for each — [[../01 - OOP Fundamentals/00 - Encapsulation]]
- [ ] All five SOLID principles with a one-sentence violation example each — [[../02 - SOLID/00 - SRP]]
- [ ] Why composition is usually preferred over inheritance — [[../01 - OOP Fundamentals/04 - Composition vs Inheritance]]
- [ ] The 6 UML relationship arrows from memory — [[../04 - UML/00 - Class Diagram]]
- [ ] Strategy vs State — same UML shape, different intent — [[../05 - Design Patterns/Behavioral/Strategy]]
- [ ] Proxy vs Decorator — same code shape, different intent — [[../05 - Design Patterns/Structural/Proxy]]
- [ ] Factory Method vs Abstract Factory vs Builder — [[../05 - Design Patterns/Creational/Factory]]
- [ ] How to make a check-then-act sequence atomic, and why it matters — [[../07 - Concurrency/05 - Race Conditions]]
- [ ] synchronized vs ReentrantLock vs AtomicInteger — when to use which — [[../07 - Concurrency/01 - synchronized]]
- [ ] The 30-minute process end to end — [[00 - 30-Minute LLD]]
- [ ] At least 3 of the LLD problems, fully, including the concurrency angle — [[../08 - LLD Problems/00 - Parking Lot]]
- [ ] How SAloon wires 4+ patterns together in one real workflow — [[../09 - Real Project LLD/SAloon/00 - Booking]]

## 🏢 Why SAloon specifically is on this list
If I can explain the SAloon booking flow end to end — entities, patterns, concurrency bug, and the fix — from memory, that's a strong signal I actually understand *why* each pattern exists, not just its definition.

## 🔥 The question I ask myself for every pattern
*"What breaks if I don't use this pattern here?"* — if I can't answer that concretely, I don't actually understand the pattern yet, I've just memorized its structure.

## 🔗 Related
[[00 - 30-Minute LLD]]
[[01 - Common Traps]]
[[03 - Design Pattern Selection]]

#interview-prep
