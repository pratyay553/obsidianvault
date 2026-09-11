---
tags: [oop]
---

# Composition vs Inheritance

## 🧠 Problem
I need a `Car` to have an `Engine`. Do I make `Car extends Engine`, or does `Car` just *hold* an `Engine`?

## 🤔 Why does the problem exist
Inheritance looked like free code reuse, so early OOP code (including my own first drafts) over-uses it. Then requirements shift — I need an electric variant, a hybrid variant — and the class hierarchy explodes or breaks LSP.

## 💡 Idea
Prefer **has-a** (composition, an object holds a reference to another object it delegates to) over **is-a** (inheritance) unless the is-a relationship is genuinely true and stable. Composition is assembled at runtime and can be swapped; inheritance is locked in at compile time.

## 🏗️ Structure
```
Inheritance:      Composition:
   Engine            Car
     ▲                └── has-a → Engine (interface)
     |
    Car
```

## ☕ Java Implementation
```java
interface Engine { void start(); }
class PetrolEngine implements Engine { public void start() {} }
class ElectricEngine implements Engine { public void start() {} }

class Car {
    private final Engine engine; // composition — swappable at construction time
    Car(Engine engine) { this.engine = engine; }
}
```
Compare: `class Car extends PetrolEngine` locks the engine type forever and makes "Car is a kind of Engine" — which is semantically wrong anyway.

## 🏢 SAloon Example
`PricingStrategy` is composed *into* `Booking`, not inherited. A booking can be re-priced (weekday → festival pricing) just by swapping the injected strategy — no subclassing `Booking` per pricing scheme. See [[05 - Design Patterns/Behavioral/Strategy]].

## 🎯 When should I use it?
Composition, by default, whenever two classes are related but might vary independently.

## ⚠️ When NOT to use it?
When the is-a relationship is genuinely permanent and the subtype never breaks the parent's contract (e.g. `Circle extends Shape` where `Shape` only defines `area()`).

## 🔥 Interview Question
*"Why is 'favor composition over inheritance' repeated so much?"*
→ Because inheritance is the tightest possible coupling (compile-time, single parent, exposes all protected internals to children), while composition is loose (runtime, swappable, interface-based) — and requirements change more often than architecture should have to.

## 🔗 Related
[[02 - Inheritance]]
[[05 - Design Patterns/Behavioral/Strategy]]
[[03 - Design Principles/03 - Law of Demeter]]

#oop
