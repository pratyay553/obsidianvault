---
tags: [lld-problem]
---

# Elevator System

## 🧠 Problem
Design a multi-elevator system that handles floor requests efficiently (minimizing direction changes) and models each elevator's door/motion state correctly.

## 🤔 Why does the problem exist
Elevators have real behavioral complexity — an idle elevator behaves differently than a moving one, and naive FCFS scheduling gives terrible average wait times. This tests whether you reach for [[../05 - Design Patterns/Behavioral/State|State]] instead of a scattered `if` on a status field.

## 💡 Idea
Each elevator's behavior lives in a `State` object (`Idle`, `Moving`, `DoorOpen`). A `Mediator`-style controller dispatches requests to whichever elevator can serve them best, using a swappable scheduling `Strategy`.

## 🏗️ Structure
```
ElevatorController (Singleton + Mediator)
  └── Elevator[]
        - state: ElevatorState (Idle/Moving/DoorOpen)
        - requestQueue: TreeSet<Integer>
SchedulingStrategy (interface) → NearestCarStrategy, ScanStrategy
```

## ☕ Java Implementation (state shape)
```java
interface ElevatorState { void handle(Elevator e); }
class MovingState implements ElevatorState {
    public void handle(Elevator e) { e.moveTowardNextRequest(); }
}
```

## 🏢 SAloon Tie-in
The `ElevatorController` (Mediator) is the same shape as any central coordinator SAloon uses to avoid two subsystems talking to each other directly — see [[../05 - Design Patterns/Behavioral/State]] used identically for `Booking`'s lifecycle in [[../09 - Real Project LLD/SAloon/00 - Booking]].

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/State]] — Idle/Moving/DoorOpen
- [[../05 - Design Patterns/Behavioral/Strategy]] — pluggable scheduling algorithm
- [[../05 - Design Patterns/Behavioral/Command]] — each floor/cabin press as a queued request

## 🔥 Interview Question
*"What's the SCAN/LOOK algorithm and why use it over FCFS?"*
→ The elevator keeps moving in its current direction, serving every request along the way, before reversing — this avoids the "elevator ping-pongs across floors" problem naive FCFS causes and roughly halves average wait time.

## 🔗 Related
[[00 - Parking Lot]]
[[../05 - Design Patterns/Behavioral/State]]

#lld-problem
