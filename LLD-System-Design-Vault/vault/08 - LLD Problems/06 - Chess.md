---
tags: [lld-problem]
---

# Chess Game

## 🧠 Problem
Design a chess game with legal-move validation per piece, check/checkmate detection, and move undo/history.

## 🤔 Why does the problem exist
This is one of the harder LLD problems because it stacks several patterns at once: each piece has its own movement rules (Strategy), moves need to be undoable (Command + Memento), and check-detection requires simulating a move without committing to it.

## 💡 Idea
Each `Piece` subclass encapsulates its own move-generation logic. Every move is a `Command` object with `execute()`/`undo()`. Before finalizing a move, simulate it and use a `Memento` snapshot to roll back if it leaves your own king in check.

## 🏗️ Structure
```
Piece (abstract) → King, Queen, Rook, Bishop, Knight, Pawn
  + getValidMoves(board): List<Square>
Move (Command) — from, to, capturedPiece; execute()/undo()
Game — board, moveHistory: List<Move>
```

## ☕ Java Implementation (move as command)
```java
class Move {
    Square from, to; Piece captured;
    void execute(Board b) { captured = b.pieceAt(to); b.movePiece(from, to); }
    void undo(Board b) { b.movePiece(to, from); if (captured != null) b.placePiece(to, captured); }
}
```

## 🏢 SAloon Tie-in
The `Move`-as-Command pattern is identical to SAloon's admin action audit trail — see [[../05 - Design Patterns/Behavioral/Command]] and [[../09 - Real Project LLD/SAloon/00 - Booking]].

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/Strategy]] — per-piece move rules
- [[../05 - Design Patterns/Behavioral/Command]] — move history, undo/redo
- [[../05 - Design Patterns/Creational/Abstract Factory]] — building a consistent piece set per variant

## 🔥 Interview Question
*"How do you detect checkmate?"*
→ King is in check AND no legal move exists that removes the check — checking the second half requires simulating every legal move for every one of the player's pieces and testing if any resolves the check, which is why undo (Memento/Command) matters for performance and correctness.

## 🔗 Related
[[05 - Tic Tac Toe]]
[[../05 - Design Patterns/Behavioral/Command]]

#lld-problem
