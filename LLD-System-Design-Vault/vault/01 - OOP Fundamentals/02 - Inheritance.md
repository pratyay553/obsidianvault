---
tags: [oop]
---

# Inheritance

## 🧠 Problem
`Employee` and `Manager` share fields like `name`, `id`, `salary`. I don't want to write that twice.

## 🤔 Why does the problem exist
Copy-pasting fields/methods across classes means every bug fix and every new field has to be applied N times. Inheritance lets me write it once in a base class.

## 💡 Idea
A subclass extends a base class and gets its fields/methods "for free," and can override behavior. Models a true **is-a** relationship.

## 🏗️ Structure
```
Employee
   ▲
   |
Manager
```

## ☕ Java Implementation
```java
class Employee {
    protected String name;
    double baseSalary() { return 30000; }
}
class Manager extends Employee {
    @Override double baseSalary() { return super.baseSalary() + 15000; }
}
```

## 🏢 SAloon Example
I actually **avoided** inheritance for `Stylist extends Employee` because a stylist and a receptionist don't behave the same way beyond sharing a name/id — I pulled the shared bit into a `StaffMember` value and composed roles instead. See [[03 - Composition vs Inheritance]] for why.

## 🎯 When should I use it?
When the subtype really can be used *anywhere* the base type is expected, with no surprising behavior (Liskov — [[02 - SOLID/02 - LSP]]), and the hierarchy is shallow (1–2 levels).

## ⚠️ When NOT to use it?
- To reuse code between classes that aren't truly "is-a" related (use composition instead).
- Deep hierarchies (3+ levels) — a change at the top ripples everywhere ("fragile base class").

## 🔥 Interview Question
*"Square extends Rectangle — good or bad design?"*
→ Bad. Setting width on a Square must also change height, which breaks any code that assumes `Rectangle.setWidth()` only changes width. Classic LSP violation.

## 🔗 Related
[[03 - Composition vs Inheritance]]
[[02 - SOLID/02 - LSP]]
[[02 - SOLID/01 - OCP]]

#oop
