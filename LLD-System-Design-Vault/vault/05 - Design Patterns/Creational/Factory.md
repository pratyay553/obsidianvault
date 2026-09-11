---
tags: [pattern, creational]
---

# Factory Method

## 🧠 Problem
SAloon needs a `Notifier` — but which concrete one (SMS vs Email) depends on a `channel` field on the customer, decided at runtime. I don't want `if/else` scattered everywhere a `Notifier` gets created.

## 🤔 Why does the problem exist
If creation logic is duplicated at every call site, adding a new channel means hunting down and editing every one of those call sites.

## 💡 Idea
Centralize "which concrete class to build" behind one factory method. Callers ask for a `Notifier` by intent, not by `new`-ing a concrete class directly.

## 🏗️ Structure
```
NotifierFactory.create(channel)
     ├── SmsNotifier
     ├── EmailNotifier
     └── PushNotifier
```

## ☕ Java Implementation
```java
interface Notifier { void send(String msg); }
class SmsNotifier implements Notifier { public void send(String m) {} }
class EmailNotifier implements Notifier { public void send(String m) {} }

class NotifierFactory {
    static Notifier create(Channel channel) {
        return switch (channel) {
            case SMS -> new SmsNotifier();
            case EMAIL -> new EmailNotifier();
        };
    }
}
```

## 🏢 SAloon Example
`NotifierFactory` is exactly this — see [[09 - Real Project LLD/SAloon/02 - Notification]]. Adding WhatsApp later meant one new class + one new `case`, nothing else changed ([[02 - SOLID/01 - OCP|OCP]] in action).

## 🎯 When should I use it?
Object creation logic is nontrivial or the concrete class depends on a runtime value.

## ⚠️ When NOT to use it?
If there's only ever one implementation and no sign of a second, a factory is a wrapper around `new` that adds nothing.

## 🔥 Interview Question
*"Factory Method vs just calling `new` directly in a `switch`?"*
→ The factory centralizes the decision in one place — if that switch logic needs to change, there's exactly one place to change it, not N scattered call sites.

## 🔗 Related
[[02 - SOLID/01 - OCP]]
[[Abstract Factory]]
[[Builder]]

#pattern/creational
