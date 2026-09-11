---
tags: [pattern, behavioral]
---

# State

## 🧠 Problem
`Booking.cancel()` needs completely different logic depending on current status — refund logic differs for `CONFIRMED` vs `PENDING`, and it should be a no-op (or error) for `COMPLETED`. The method was turning into a `switch` on `status` with a case for every method.

## 🤔 Why does the problem exist
A status field checked via `switch` inside every method scatters the same state-transition rules across many methods — easy to update one switch and forget another, letting an illegal transition slip through.

## 💡 Idea
Each state becomes its own class implementing a shared interface. The object holds a reference to its *current state object* and delegates behavior to it — transitioning states means swapping that reference.

## 🏗️ Structure
```
BookingState (interface)
   ├── PendingState
   ├── ConfirmedState
   ├── CompletedState   (cancel() here throws)
   └── CancelledState
Booking --delegates to--> current: BookingState
```

## ☕ Java Implementation
```java
interface BookingState { void cancel(Booking b); }

class PendingState implements BookingState {
    public void cancel(Booking b) { b.setState(new CancelledState()); }
}
class CompletedState implements BookingState {
    public void cancel(Booking b) { throw new IllegalStateException("cannot cancel a completed booking"); }
}

class Booking {
    private BookingState state = new PendingState();
    void setState(BookingState s) { this.state = s; }
    void cancel() { state.cancel(this); }
}
```

## 🏢 SAloon Example
This is the literal implementation behind [[../../04 - UML/02 - State Diagram]]'s SAloon booking lifecycle — see [[09 - Real Project LLD/SAloon/00 - Booking]]. `CompletedState.cancel()` throwing is what makes an illegal transition structurally impossible, not just "checked for."

## 🎯 When should I use it?
An object's behavior depends heavily on a status field with several values, each with distinct valid transitions.

## ⚠️ When NOT to use it?
A simple boolean flag (active/inactive) doesn't need a full State pattern — that's over-engineering a 2-value switch.

## 🔥 Interview Question
*"Strategy vs State — same UML shape, what's actually different in intent?"*
→ Strategy: client picks the algorithm. State: the object itself decides to transition based on its own logic in response to events — the client just calls methods and the state changes happen internally.

## 🔗 Related
[[../../04 - UML/02 - State Diagram]]
[[Strategy]]

#pattern/behavioral
