---
tags: [lld-problem]
---

# Vending Machine

## 🧠 Problem
Design a vending machine: insert coins, select an item, dispense it and return change — while rejecting invalid actions (selecting before paying, inserting coins after dispensing).

## 🤔 Why does the problem exist
This is the textbook problem for the [[../05 - Design Patterns/Behavioral/State|State pattern]] because the same button press means something different depending on where you are in the flow — "select item" before paying should say "insert coin first," not silently do nothing or crash.

## 💡 Idea
Model each stage of the transaction as its own state class implementing a shared interface; the machine delegates every action to its current state.

## 🏗️ Structure
```
VendingState (interface) → IdleState, HasCoinState, DispensingState
VendingMachine — delegates insertCoin()/selectItem() to current state
```

## ☕ Java Implementation
```java
interface VendingState {
    void insertCoin(VendingMachine m);
    void selectItem(VendingMachine m);
}
class IdleState implements VendingState {
    public void insertCoin(VendingMachine m) { m.setState(m.hasCoinState); }
    public void selectItem(VendingMachine m) { System.out.println("Insert coin first"); }
}
```

## 🏢 SAloon Tie-in
Exactly the same shape as `Booking`'s `PendingState`/`ConfirmedState` — see [[../05 - Design Patterns/Behavioral/State]] and [[../09 - Real Project LLD/SAloon/00 - Booking]].

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/State]] — the whole point of the problem
- [[02 - ATM]]'s denomination chain reused for change-dispensing

## 🔥 Interview Question
*"How would you extend this for multiple item slots with different stock counts?"*
→ Give each slot its own quantity counter; `DispensingState` checks stock before allowing selection — the state machine doesn't need to change, just the data it checks against.

## 🔗 Related
[[02 - ATM]]
[[../05 - Design Patterns/Behavioral/State]]

#lld-problem
