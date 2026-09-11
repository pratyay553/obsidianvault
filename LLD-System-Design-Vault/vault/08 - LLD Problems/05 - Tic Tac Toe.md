---
tags: [lld-problem]
---

# Tic-Tac-Toe

## 🧠 Problem
Design a 2-player tic-tac-toe game with win/draw detection, ideally generalized to an N×N board with K-in-a-row to win.

## 🤔 Why does the problem exist
It's a good warm-up because it's small enough to fully finish in an interview, but the "generalize to N×N, K-in-a-row" follow-up tests whether your win-check is O(1) incremental or a wasteful O(n²) full-board rescan every move.

## 💡 Idea
Keep the board dumb (just grid state); put win-checking in its own pluggable `WinningStrategy` so the board doesn't need to know how "win" is defined. Track running row/column/diagonal counts incrementally instead of rescanning.

## 🏗️ Structure
```
Board — grid: Symbol[n][n]
WinningStrategy (interface) → LineWinStrategy
Game — players, currentPlayerIdx, board
```

## ☕ Java Implementation (incremental win-check)
```java
class Board {
    int[] rowCount, colCount; int diag, antiDiag;
    boolean placeAndCheckWin(int r, int c, int playerSign) {
        rowCount[r] += playerSign; colCount[c] += playerSign;
        if (r == c) diag += playerSign;
        if (r + c == size - 1) antiDiag += playerSign;
        return Math.abs(rowCount[r]) == size || Math.abs(colCount[c]) == size
            || Math.abs(diag) == size || Math.abs(antiDiag) == size;
    }
}
```

## 🏢 SAloon Tie-in
Not a natural fit for SAloon — this problem's value is purely as an OOP-fundamentals + Strategy warm-up before tackling [[06 - Chess]].

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/Strategy]] — swappable win condition

## 🔥 Interview Question
*"Generalize to an N×N board, K-in-a-row to win, not just full row/col."*
→ The incremental counters above need adjustment to track sliding windows of length K rather than the whole row — but the core idea (avoid O(n²) rescans) stays the same.

## 🔗 Related
[[06 - Chess]]
[[../05 - Design Patterns/Behavioral/Strategy]]

#lld-problem
