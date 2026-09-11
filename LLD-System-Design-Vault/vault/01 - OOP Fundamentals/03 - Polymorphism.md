---
tags: [oop]
---

# Polymorphism

## 🧠 Problem
I need to send a notification, but the channel (SMS, email, push) is decided at runtime. I don't want `if (channel == SMS) ... else if (channel == EMAIL) ...` scattered across the codebase.

## 🤔 Why does the problem exist
Type-checking `if/else` or `switch` on a type field has to be repeated at every call site, and every time a new channel is added, I have to hunt down and edit every one of those switch statements.

## 💡 Idea
One interface, many implementations. The caller invokes the same method name; the correct implementation runs based on the object's actual runtime type (dynamic dispatch).

## 🏗️ Structure
```
Notifier (interface)
   ├── SmsNotifier
   ├── EmailNotifier
   └── PushNotifier
```

## ☕ Java Implementation
```java
interface Notifier { void send(String msg); }
class SmsNotifier implements Notifier { public void send(String m) { /* twilio */ } }
class EmailNotifier implements Notifier { public void send(String m) { /* smtp */ } }

void notifyUser(Notifier n, String msg) { n.send(msg); } // doesn't care which one
```

## 🏢 SAloon Example
`Notifier` is exactly this in [[09 - Real Project LLD/SAloon/02 - Notification]] — booking confirmations go out over whichever channel the customer picked, and `BookingService` never has a single `if`.

## 🎯 When should I use it?
Any time behavior needs to vary by type and new types will keep being added — this is the mechanism under [[05 - Design Patterns/Behavioral/Strategy]], [[05 - Design Patterns/Behavioral/Observer]], and [[05 - Design Patterns/Behavioral/State]].

## ⚠️ When NOT to use it?
If there are only ever going to be 2 fixed cases and they'll never grow, a plain `if/else` is sometimes more readable than a whole interface hierarchy — don't over-engineer.

## 🔥 Interview Question
*"Compile-time vs runtime polymorphism?"*
→ Overloading (same name, different signature) resolves at compile time. Overriding (subclass redefines a method) resolves at runtime via the object's actual type — this is the one design patterns rely on.

## 🔗 Related
[[02 - Inheritance]]
[[05 - Design Patterns/Behavioral/Strategy]]
[[02 - SOLID/01 - OCP]]

#oop
