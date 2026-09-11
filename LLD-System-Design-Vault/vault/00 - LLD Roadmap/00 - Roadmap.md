---
tags: [roadmap, moc]
---

# 🧭 LLD Roadmap

I keep forgetting where I left off, so this is the fixed order I move through the vault. Don't skip ahead to Design Patterns before OOP + SOLID actually feel automatic — that's the mistake I made the first time and every pattern note stopped making sense.

## The sequence
1. [[01 - OOP Fundamentals/00 - Encapsulation|OOP Fundamentals]] — the four pillars, not just definitions, but *why* each one exists
2. [[02 - SOLID/00 - SRP|SOLID]] — the rules that tell me when a class has gone wrong
3. [[03 - Design Principles/00 - DRY|Design Principles]] — DRY, KISS, YAGNI, Law of Demeter — the "taste" layer
4. [[04 - UML/00 - Class Diagram|UML]] — enough to whiteboard a design fast
5. [[05 - Design Patterns/Creational/Singleton|Design Patterns]] — Creational → Structural → Behavioral
6. [[07 - Concurrency/00 - Thread Safety|Concurrency]] — because half of LLD interviews ask "what if two threads hit this at once"
7. [[08 - LLD Problems/00 - Parking Lot|LLD Problems]] — apply everything above under time pressure
8. [[09 - Real Project LLD/SAloon/00 - Booking|SAloon]] — one project, designed end to end, patterns wired together like they'd actually be in a codebase
9. [[10 - Interview Revision/00 - 30-Minute LLD|Interview Revision]] — the day-before-interview pass

## Why SAloon as the running example
Every pattern note in this vault also shows how it shows up in **SAloon** — a salon booking system designed fully in section 09 (booking, availability, payment, notification, pricing, review). Seeing the *same* pattern reused across a real domain sticks better than five unrelated toy examples.

## Rule for every note in this vault
No note is allowed to just define a term. Every note answers:
- 🧠 What problem am I solving
- 🤔 Why does the problem even exist (what breaks without the idea)
- 💡 The idea itself
- 🏗️ Structure
- ☕ Code
- 🏢 SAloon tie-in
- 🎯 / ⚠️ When to use / not use
- 🔥 An interview question I should be able to answer cold
- 🔗 Related notes

#roadmap #moc
