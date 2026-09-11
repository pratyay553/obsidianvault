---
tags: [principle]
---

# KISS — Keep It Simple, Stupid

## 🧠 Problem
I once wrapped `AvailabilityChecker` in a `Strategy` + `Factory` + `Observer` combo for SAloon before there was even a second availability rule to justify it. It took longer to read than to write.

## 🤔 Why does the problem exist
It's tempting to reach for a pattern because it's "correct architecture," but every layer of indirection has a real cost: more files to open, more jumps to trace one code path, more onboarding time.

## 💡 Idea
Prefer the simplest design that solves the actual current problem. Add complexity only when a *real*, *current* requirement demands it — not a hypothetical future one (pairs with [[02 - YAGNI]]).

## 🏗️ Structure
```
Simple version:                Over-engineered version:
AvailabilityChecker.isFree()   AvailabilityCheckerFactory
                                 → AvailabilityStrategy (interface)
                                   → DefaultAvailabilityStrategy
                                 → AvailabilityObserver
```

## ☕ Java Implementation
```java
class AvailabilityChecker {
    boolean isFree(Slot s, List<Booking> existing) {
        return existing.stream().noneMatch(b -> b.overlaps(s));
    }
}
```

## 🏢 SAloon Example
`AvailabilityChecker` in [[09 - Real Project LLD/SAloon/00 - Booking]] stayed a plain class with one method for months — I only introduced a `Strategy` once staff-specific buffer-time rules actually showed up.

## 🎯 When should I use it?
Constantly, as a check before reaching for a pattern — "do I have evidence I need this flexibility *today*?"

## ⚠️ When NOT to use it?
Don't use "keep it simple" as an excuse to skip validation or error handling — simple ≠ sloppy.

## 🔥 Interview Question
*"How do you avoid over-engineering in an LLD interview?"*
→ Solve the stated requirements with the plainest OOP design first, then explicitly say "if requirement X came up, I'd introduce Strategy/Factory here."

## 🔗 Related
[[00 - DRY]]
[[02 - YAGNI]]

#principle
