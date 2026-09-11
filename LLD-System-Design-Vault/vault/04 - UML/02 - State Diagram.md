---
tags: [uml]
---

# State Diagram

## 🧠 Problem
`Booking` can be `PENDING`, `CONFIRMED`, `CANCELLED`, `COMPLETED`, `NO_SHOW`. Not every transition is legal — a `COMPLETED` booking should never go back to `PENDING`.

## 🤔 Why does the problem exist
Status fields left unconstrained let any code set `booking.status = ANYTHING`, quietly allowing illegal transitions unless every call site independently remembers the rules.

## 💡 Idea
Draw states as nodes, legal transitions as labeled edges. Anything not drawn is **not allowed** — this becomes the direct spec for a [[05 - Design Patterns/Behavioral/State|State pattern]] implementation.

## 🏗️ Structure
```
PENDING --pay()--> CONFIRMED --complete()--> COMPLETED
   |                   |
   cancel()          cancel()
   ↓                   ↓
CANCELLED           CANCELLED
```
`COMPLETED` and `CANCELLED` have no outgoing edges — terminal states.

## 🏢 SAloon Example
This is the literal state diagram for [[09 - Real Project LLD/SAloon/00 - Booking]] — implemented with the [[05 - Design Patterns/Behavioral/State|State pattern]] so `Booking.cancel()` throws if called from `COMPLETED`.

## 🎯 When should I use it?
Any entity with a status/lifecycle field with more than 2-3 states and non-trivial transition rules.

## ⚠️ When NOT to use it?
A boolean flag doesn't need a state diagram.

## 🔥 Interview Question
*"How do you stop invalid state transitions in code, not just on paper?"*
→ Each state becomes its own class that only exposes the transitions valid *from that state* — see [[05 - Design Patterns/Behavioral/State]].

## 🔗 Related
[[05 - Design Patterns/Behavioral/State]]
[[00 - Class Diagram]]

#uml
