---
tags: [uml]
---

# Class Diagram

## 🧠 Problem
I need to explain a design to a teammate (or an interviewer) faster than reading through actual class files.

## 🤔 Why does the problem exist
Prose descriptions of "which class has which field and talks to what" are slow to parse and easy to describe inconsistently. A diagram with fixed notation removes ambiguity.

## 💡 Idea
A class diagram is a static snapshot: boxes for classes, arrows for relationships. Learn the six arrow types cold — that's 90% of what's needed in an interview.

## 🏗️ Structure
```
+----------------------+
|      ClassName       |
+----------------------+
| - privateField: Type |
+----------------------+
| + publicMethod(): T  |
+----------------------+
```

## ☕ The arrows, memorized
| Relationship | Notation | Meaning |
|---|---|---|
| Inheritance | `──▷` hollow triangle, solid | is-a |
| Realization | `┈┈▷` hollow triangle, dashed | implements interface |
| Composition | `◆──` filled diamond | strong has-a, owns lifecycle |
| Aggregation | `◇──` hollow diamond | weak has-a, independent lifecycle |
| Association | `───` plain | uses/knows about |
| Dependency | `┈┈>` dashed arrow | temporary use (method param) |

## 🏢 SAloon Example
```
Booking ◆── Slot           (composition — a slot belongs to exactly one booking's lifetime)
Booking ── Customer        (association — booking knows a customer, customer outlives it)
Booking ┈┈> PricingStrategy (dependency — used at calculation time)
```
See the full diagram in [[09 - Real Project LLD/SAloon/00 - Booking]].

## 🎯 When should I use it?
At the start of any LLD interview answer — sketch the core entities and relationships in plain text before writing a line of Java.

## ⚠️ When NOT to use it?
Don't spend interview time perfecting arrow notation — plain-text pseudo-UML is what most interviewers actually want.

## 🔥 Interview Question
*"Composition or aggregation between Order and OrderItem?"*
→ Composition — an `OrderItem` has no meaning/lifecycle outside its `Order`. Compare to `Order` and `Customer` — aggregation, the customer exists independently.

## 🔗 Related
[[01 - OOP Fundamentals/04 - Composition vs Inheritance]]
[[01 - Sequence Diagram]]

#uml
