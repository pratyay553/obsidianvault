---
tags: [pattern, creational]
---

# Singleton

## 🧠 Problem
SAloon needs exactly one `AppointmentScheduler` coordinating all bookings across the app — two separate instances would each think they have full visibility into slot availability and double-book the same chair.

## 🤔 Why does the problem exist
If any code can `new AppointmentScheduler()`, nothing stops two instances from existing with two separate, inconsistent views of "what's booked."

## 💡 Idea
Make the constructor private, hide the one instance behind a static accessor. There is provably only ever one.

## 🏗️ Structure
```
AppointmentScheduler
  - private static instance
  - private constructor
  + static getInstance()
```

## ☕ Java Implementation
```java
public class AppointmentScheduler {
    private static volatile AppointmentScheduler instance;
    private AppointmentScheduler() {}

    public static AppointmentScheduler getInstance() {
        if (instance == null) {
            synchronized (AppointmentScheduler.class) {
                if (instance == null) instance = new AppointmentScheduler();
            }
        }
        return instance;
    }
}
```
Double-checked locking so we're not synchronizing on every call — see [[07 - Concurrency/01 - synchronized]].

## 🏢 SAloon Example
`AppointmentScheduler` — see [[09 - Real Project LLD/SAloon/00 - Booking]]. Also used for the central `SlotLockManager`.

## 🎯 When should I use it?
A genuinely single shared resource: config manager, connection pool, central scheduler/lock manager, logger.

## ⚠️ When NOT to use it?
I almost singletoned `PricingService` too — wrong call. It has no shared mutable state, so it should just be dependency-injected as a normal object. Singleton is hidden global state, which made unit testing painful — I had to manually reset static state between tests.

## 🔥 Interview Question
*"What's wrong with Singleton, if it's so useful?"*
→ It's global mutable state — hides dependencies and makes tests need manual reset/mocking tricks. Prefer DI with the container guaranteeing a single instance where possible.

## 🔗 Related
[[02 - SOLID/04 - DIP]]
[[07 - Concurrency/01 - synchronized]]

#pattern/creational
