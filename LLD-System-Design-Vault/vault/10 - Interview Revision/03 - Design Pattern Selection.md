---
tags: [interview-prep]
---

# Design Pattern Selection — Decision Guide

## 🧠 Problem
Mid-interview, staring at a requirement, it's easy to blank on "which pattern fits this" or worse, force-fit the wrong one.

## 🤔 Why a decision guide helps
Patterns are easy to memorize in isolation but hard to *retrieve* under pressure when you're looking at a fresh problem statement instead of a pattern name. A trigger-phrase mapping closes that gap.

## 💡 The mapping (trigger phrase in the requirement → pattern)

| If the requirement says... | Reach for... |
|---|---|
| "different algorithms/rules, chosen at runtime" | [[../05 - Design Patterns/Behavioral/Strategy]] |
| "object behaves differently depending on its status" | [[../05 - Design Patterns/Behavioral/State]] |
| "notify multiple parts of the system when X happens" | [[../05 - Design Patterns/Behavioral/Observer]] |
| "need undo, audit log, or a job queue" | [[../05 - Design Patterns/Behavioral/Command]] |
| "request passes through several checks/validators" | [[../05 - Design Patterns/Behavioral/Chain of Responsibility]] |
| "many optional fields when constructing X" | [[../05 - Design Patterns/Creational/Builder]] |
| "which concrete class depends on a runtime value" | [[../05 - Design Patterns/Creational/Factory]] |
| "need a matching/compatible *family* of objects" | [[../05 - Design Patterns/Creational/Abstract Factory]] |
| "exactly one shared instance across the app" | [[../05 - Design Patterns/Creational/Singleton]] |
| "integrate a third-party API with a different shape" | [[../05 - Design Patterns/Structural/Adapter]] |
| "simplify access to a complex multi-step workflow" | [[../05 - Design Patterns/Structural/Facade]] |
| "add optional, combinable behavior to individual objects" | [[../05 - Design Patterns/Structural/Decorator]] |
| "control/restrict access, or lazy-load something expensive" | [[../05 - Design Patterns/Structural/Proxy]] |

## The check I run before committing to a pattern
1. Does the *stated* requirement actually need this, or am I predicting a future one? (→ [[../03 - Design Principles/02 - YAGNI]])
2. Is there a simpler design that still satisfies the requirement? (→ [[../03 - Design Principles/01 - KISS]])
3. Can I point to the exact sentence in the requirements that justifies this pattern?

If the answer to #3 is "no, I just think it's good practice here" — that's the signal to not add it yet, and instead mention it as a "if requirement X came up, I'd introduce Y here."

## 🏢 SAloon Example
[[../09 - Real Project LLD/SAloon/04 - Pricing]] is the clearest example of applying this table live: "mutually exclusive base rate" → Strategy; "stackable modifiers" → Decorator — each mapped directly from a specific requirement sentence, not picked speculatively.

## 🔥 Interview Question
*"How do you avoid over-engineering while still showing pattern knowledge?"*
→ Solve the stated requirement with the simplest correct OOP design, then explicitly narrate: "If requirement X came up next, I'd introduce pattern Y here, because..." — this demonstrates knowledge without inflating the current design.

## 🔗 Related
[[00 - 30-Minute LLD]]
[[../03 - Design Principles/02 - YAGNI]]

#interview-prep
