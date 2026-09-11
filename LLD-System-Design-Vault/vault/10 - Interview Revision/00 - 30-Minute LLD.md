---
tags: [interview-prep]
---

# The 30-Minute LLD Approach

## 🧠 Problem
In an actual interview, I don't have time to fully design something the way SAloon was designed over months — I have ~30 minutes and need a repeatable process that produces a defensible design fast.

## 🤔 Why does the problem exist
Without a fixed process, it's easy to either dive straight into code (and design yourself into a corner) or spend too long gathering requirements and run out of time to write anything.

## 💡 Idea
A fixed time-boxed sequence: clarify → identify entities → sketch relationships → identify what varies → apply patterns only where variation exists → code the core, not everything.

## 🏗️ The actual sequence I use
```
0-5 min   Clarify requirements out loud — ask 3-5 pointed questions
5-10 min  List core entities/nouns from the problem statement
10-15 min Sketch a class diagram (plain text is fine) — relationships between entities
15-20 min Identify "what varies" — this tells you which design patterns actually apply
20-28 min Write the core classes in code — don't implement every method, focus on signatures + the interesting ones
28-30 min State trade-offs and what you'd add given more time (concurrency, error handling, scaling)
```

## The "what varies" checklist (this is the actual skill)
- Multiple ways to do the same computation, chosen at runtime → [[../05 - Design Patterns/Behavioral/Strategy]]
- An entity's behavior changes by status/lifecycle stage → [[../05 - Design Patterns/Behavioral/State]]
- Object needs many optional fields → [[../05 - Design Patterns/Creational/Builder]]
- Need to notify multiple unrelated parts of the system on an event → [[../05 - Design Patterns/Behavioral/Observer]]
- Need undo/audit/queueing of actions → [[../05 - Design Patterns/Behavioral/Command]]
- Integrating a third-party API with a different shape → [[../05 - Design Patterns/Structural/Adapter]]
- Coordinating many services for one workflow → [[../05 - Design Patterns/Structural/Facade]]
- Adding optional, combinable behavior to individual objects → [[../05 - Design Patterns/Structural/Decorator]]

## 🏢 SAloon Example
Every module in section 09 followed this same loop — [[../09 - Real Project LLD/SAloon/04 - Pricing]] is a good compressed example: "what varies" was identified as (a) base rate — mutually exclusive → Strategy, and (b) modifiers — stackable → Decorator.

## 🔥 The meta-interview-question
*"How do you know when to stop adding patterns?"*
→ When you run out of *actual, stated* variation points. See [[03 - Design Pattern Selection]] and [[../03 - Design Principles/02 - YAGNI]] — don't apply a pattern speculatively; every pattern should trace back to a specific requirement in the problem statement.

## 🔗 Related
[[01 - Common Traps]]
[[03 - Design Pattern Selection]]

#interview-prep
