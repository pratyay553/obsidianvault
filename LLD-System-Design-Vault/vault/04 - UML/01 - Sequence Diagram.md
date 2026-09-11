---
tags: [uml]
---

# Sequence Diagram

## 🧠 Problem
A class diagram shows *what* objects exist and how they're related — but not the *order* of calls during a booking checkout, and that order is where the tricky bugs live.

## 🤔 Why does the problem exist
Static structure alone can't answer "does payment happen before or after the slot is locked?" — that's a question about time and message order.

## 💡 Idea
Show objects as vertical lifelines, and arrows between them in time order (top to bottom) — each arrow is a method call/message.

## 🏗️ Structure
```
Customer      BookingService      SlotRepo      PaymentGateway
   |                |                 |               |
   |--book()------->|                 |               |
   |                |--lockSlot()---->|               |
   |                |<---ok-----------|               |
   |                |--charge()------------------------>|
   |                |<-------------------------success--|
   |<--confirmed----|                 |               |
```

## 🏢 SAloon Example
This exact sequence is how I caught a bug: I originally charged the customer *before* locking the slot — meaning two customers could pay for the same slot in a race. Drawing this made the ordering bug obvious. See [[09 - Real Project LLD/SAloon/00 - Booking]] and [[07 - Concurrency/05 - Race Conditions]].

## 🎯 When should I use it?
Whenever a flow involves more than 2 objects, or when ordering/timing matters.

## ⚠️ When NOT to use it?
Don't bother for a single object with a single method call.

## 🔥 Interview Question
*"Walk me through what happens, in order, when two customers try to book the same slot at the same time."*
→ Exactly what a sequence diagram answers — draw it, and the fix (lock before charge) becomes visually obvious.

## 🔗 Related
[[00 - Class Diagram]]
[[07 - Concurrency/05 - Race Conditions]]

#uml
