---
tags: [interview-prep]
---

# Common LLD Interview Traps

## 🧠 Problem
Certain mistakes show up over and over in LLD interviews (mine included, on the way to actually learning this) — worth having as an explicit checklist to avoid on the day.

## 🤔 Why these keep happening
Most come from reaching for familiar tools (inheritance, getters/setters, one big service class) before checking whether they actually fit the requirement — habits from earlier, less disciplined code.

## The traps, in the order I actually hit them learning this

**1. Anemic domain model** — classes are just data + getters/setters, all logic lives in a separate `*Service`. See [[../01 - OOP Fundamentals/00 - Encapsulation]] — if a class can describe its own rules (`canCancel()`), let it.

**2. Inheritance instead of composition** — modeling `WeekendStylist extends Stylist` for what should be an injected `PricingStrategy`. See [[../01 - OOP Fundamentals/04 - Composition vs Inheritance]].

**3. LSP violations from convenience inheritance** — a subclass that throws `UnsupportedOperationException` to "opt out" of an inherited method. See [[../02 - SOLID/02 - LSP]].

**4. Pattern-first instead of requirement-first design** — introducing `Strategy`/`Factory`/`Observer` because they feel like "good architecture," before there's an actual second implementation to justify it. See [[../03 - Design Principles/02 - YAGNI]].

**5. Ignoring concurrency entirely** — designing a booking/allocation system and never mentioning what happens if two requests race for the same resource. Interviewers often ask this as a follow-up specifically to see if it was considered. See [[../07 - Concurrency/05 - Race Conditions]].

**6. Fat interfaces** — one `StaffMember` interface forcing every role to implement methods that don't apply. See [[../02 - SOLID/03 - ISP]].

**7. God classes / one class doing orchestration AND business logic AND persistence** — see [[../02 - SOLID/00 - SRP]] and use a [[../05 - Design Patterns/Structural/Facade]] for orchestration specifically.

**8. Skipping the class diagram and jumping straight to code** — leads to redesigning mid-interview when a missed relationship surfaces. See [[../04 - UML/00 - Class Diagram]].

## 🏢 SAloon Example
Traps #1, #2, #3, and #5 are all traps I hit *for real* while building SAloon — see [[../09 - Real Project LLD/SAloon/01 - Payment]] (LSP), [[../09 - Real Project LLD/SAloon/00 - Booking]] (concurrency), and [[../01 - OOP Fundamentals/02 - Inheritance]] (composition).

## 🔥 Interview Question
*"What's the most common mistake you see (or made) in LLD design?"*
→ Anemic domain models — pulling every bit of behavior out into a `Service` layer and leaving entities as pure data, which erases the whole point of OOP encapsulation.

## 🔗 Related
[[00 - 30-Minute LLD]]
[[../01 - OOP Fundamentals/00 - Encapsulation]]

#interview-prep
