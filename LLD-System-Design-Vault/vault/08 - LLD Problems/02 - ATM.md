---
tags: [lld-problem]
---

# ATM Machine

## 🧠 Problem
Design an ATM that handles card insertion, PIN validation, balance check, and cash withdrawal — where withdrawal has to dispense the fewest possible notes and never dispense more cash than available.

## 🤔 Why does the problem exist
Like [[04 - Vending Machine]], the ATM's behavior genuinely differs by stage (no card inserted vs card inserted vs authenticated) — trying to guard all of this with boolean flags gets unreadable fast, and cash-dispensing has a real algorithmic sub-problem (greedy denomination selection).

## 💡 Idea
Model the session as a `State` machine (`Idle`, `CardInserted`, `Authenticated`), and model each cash tray as a `Denomination` handler in a `Chain of Responsibility` — hand off dispensing to the largest denomination first, then the next, etc.

## 🏗️ Structure
```
ATMState (interface) → IdleState, CardInsertedState, AuthenticatedState
CashDispenser
  └── DenominationHandler chain: 2000 → 500 → 100 (Chain of Responsibility)
```

## ☕ Java Implementation (dispensing chain)
```java
abstract class DenominationHandler {
    protected DenominationHandler next;
    abstract int dispense(int amount, Map<Integer, Integer> trayCounts);
}
class Note2000Handler extends DenominationHandler {
    int dispense(int amount, Map<Integer, Integer> trays) {
        int count = Math.min(amount / 2000, trays.get(2000));
        trays.put(2000, trays.get(2000) - count);
        int remaining = amount - count * 2000;
        return remaining > 0 && next != null ? next.dispense(remaining, trays) : remaining;
    }
}
```

## 🏢 SAloon Tie-in
The `ATMState` machine is structurally identical to [[../09 - Real Project LLD/SAloon/00 - Booking]]'s `BookingState` — same [[../05 - Design Patterns/Behavioral/State|State pattern]], different domain.

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/State]] — session states
- [[../05 - Design Patterns/Behavioral/Chain of Responsibility]] — denomination dispensing
- [[../05 - Design Patterns/Behavioral/Command]] — transaction logging/undo on failed dispense

## 🔥 Interview Question
*"What happens if the machine runs out of a denomination mid-dispense?"*
→ The chain should fall through to the next smaller denomination automatically (as coded above) — and the whole withdrawal should be wrapped in a transaction so a partial dispense failure doesn't leave the account debited without cash out.

## 🔗 Related
[[04 - Vending Machine]]
[[../05 - Design Patterns/Behavioral/State]]

#lld-problem
